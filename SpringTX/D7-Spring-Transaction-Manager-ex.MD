프로그래밍적 트랜잭션 관리는 선언적 방식만으로는 해결하기 어렵거나, 트랜잭션 경계를 동적으로 제어해야 할 때 주로 사용됩니다. 구체적인 사례를 몇 가지 들어보겠습니다.

---

## 1. 대용량 배치 처리(Chunk 단위 커밋)

**상황**
수십만 건의 데이터를 한 번에 처리하면 메모리·DB 락 유지 부담이 커집니다.
**해결 포인트**
일정 단위(예: 1,000건)마다 커밋하여 메모리와 락을 해제하고, 실패 시 해당 청크만 롤백.

```kotlin
fun importLargeData(records: List<Record>) {
    val chunkSize = 1000
    records.chunked(chunkSize).forEachIndexed { idx, chunk ->
        txTemplate.execute<Void> { status ->
            chunk.forEach { repo.save(it) }
            println("Chunk ${idx + 1} committed.")
            null
        }
    }
}
```

---

## 2. 동적 트랜잭션 속성 적용

**상황**
고객 VIP 여부나 서비스 종류에 따라 다른 격리 수준(또는 타임아웃)을 적용해야 할 때.
**해결 포인트**
런타임에 `isolationLevel`이나 `timeout`을 변경하여 트랜잭션 실행.

```kotlin
fun processOrder(order: Order, vip: Boolean) {
    // VIP는 좀 더 높은 격리 수준, 일반은 디폴트
    txTemplate.isolationLevel = if (vip) 
        TransactionDefinition.ISOLATION_SERIALIZABLE 
    else 
        TransactionDefinition.ISOLATION_READ_COMMITTED

    txTemplate.execute<Void> {
        orderRepo.save(order)
        null
    }
}
```

---

## 3. 논리적 경계 분리(커밋 직후 무거운 연산)

**상황**
주문 저장 → 결제 승인 → 외부 시스템 연동(무거운 연산)이 순차적으로 실행되는데, 외부 호출 실패 시라도 주문 저장 결과만큼은 확정시키고 싶을 때.
**해결 포인트**
첫 번째 트랜잭션에서 저장·커밋 후, 그 결과를 바탕으로 두 번째 트랜잭션을 분리 실행.

```kotlin
fun placeOrderWithAsyncTasks(order: Order) {
    // 1) 주문 저장만 커밋
    txTemplate.execute<Order> {
        orderRepo.save(order)
    }?.let { savedOrder ->
        // 2) 별도 트랜잭션: 외부 연동
        txTemplate.execute<Void> {
            externalService.notify(savedOrder)
            null
        }
    }
}
```

---

## 4. Spring 관리 범위 밖 코드에서 트랜잭션

**상황**
스프링 빈이 아닌 헬퍼(유틸) 클래스나 라이브러리 코드에서 트랜잭션이 필요할 때.
**해결 포인트**
`TransactionTemplate` 을 직접 주입받아 사용하면 AOP 프록시 없이도 트랜잭션 적용 가능.

```kotlin
class AuditUtil(private val txTemplate: TransactionTemplate,
                private val auditRepo: AuditRepository) {
    fun auditAction(entry: AuditEntry) {
        txTemplate.execute<Void> {
            auditRepo.insert(entry)
            null
        }
    }
}
```

---

## 5. 멀티 데이터소스 간 분기 처리

**상황**
A DB에는 주문, B DB에는 통계 데이터를 저장하는데, 상황에 따라 B만 롤백하거나 A만 롤백해야 할 때.
**해결 포인트**
각각의 `TransactionTemplate`(A용, B용)을 분리하여, 필요에 따라 개별 트랜잭션을 제어.

```kotlin
class MultiDbService(
    private val txA: TransactionTemplate,
    private val txB: TransactionTemplate,
    private val orderRepo: OrderRepo,
    private val statsRepo: StatsRepo
) {
    fun handle(order: Order) {
        txA.execute<Void> {
            orderRepo.save(order)
            null
        }
        txB.execute<Void> { status ->
            statsRepo.record(order)
            // 조건에 따라 B만 롤백
            if (order.amount > MAX) status.setRollbackOnly()
            null
        }
    }
}
```

---

## 6. 반복 재시도 로직(Retry)

**상황**
트랜잭션 충돌(Deadlock) 발생 시 자동으로 재시도하고 싶을 때.
**해결 포인트**
`execute`를 래핑해 재시도 루프를 구현.

```kotlin
fun <T> withRetry(maxAttempts: Int, block: (TransactionStatus) -> T): T {
    repeat(maxAttempts - 1) {
        try {
            return txTemplate.execute(block)!!
        } catch (ex: CannotAcquireLockException) {
            // 잠시 대기 후 재시도
            Thread.sleep(100)
        }
    }
    // 마지막 시도
    return txTemplate.execute(block)!!
}
```

---

## 언제 프로그래밍적 트랜잭션을 고려할까?

1. **트랜잭션 경계를 동적으로 바꿔야 할 때** (격리 수준, 타임아웃 등)
2. **일부 작업만 커밋/롤백 제어가 필요할 때** (논리적 분리)
3. **루프/배치 중 청크 단위 커밋이 필요할 때**
4. **Spring 관리 밖 코드에서 트랜잭션을 걸어야 할 때**
5. **멀티 데이터소스 또는 분기별 분리 관리가 필요할 때**
6. **커스텀 재시도·회복 로직을 넣어야 할 때**

대부분의 일반 CRUD·서비스 단위 로직은 선언적 관리로 충분하지만, 위와 같은 **동적·분리·재시도·외부 통제** 시나리오에서는 프로그래밍적 관리가 강력한 무기가 됩니다.
