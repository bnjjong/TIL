## ✅ 1. 전파(Propagation)란?

Spring에서 트랜잭션은 AOP 기반의 프록시 방식으로 관리됩니다. `@Transactional`이 선언된 메서드가 호출될 때 현재 트랜잭션의 존재 여부에 따라 다음과 같은 전략을 지정할 수 있으며, 이것을 "전파(Propagation)" 속성이라 부릅니다.

전파 속성은 다음의 7가지로 구분됩니다:

| Propagation    | 설명                                                |
| -------------- | ------------------------------------------------- |
| REQUIRED       | (기본값) 현재 트랜잭션이 있으면 참여, 없으면 `새로 생성`                  |
| REQUIRES\_NEW  | `항상 새 트랜잭션 생성`, 기존 트랜잭션은 일시 정지                      |
| NESTED         | 현재 트랜잭션 내부에 중첩 트랜잭션 생성 (savepoint 기반 rollback 지원) |
| MANDATORY      | 반드시 트랜잭션 존재 필요, 없으면 예외 발생                         |
| NEVER          | 트랜잭션 있으면 예외 발생                                    |
| NOT\_SUPPORTED | 트랜잭션이 있으면 일시 중단하고 트랜잭션 없이 실행                      |
| SUPPORTS       | 트랜잭션이 있으면 참여, 없으면 비트랜잭션으로 실행                      |

---

## ✅ 2. 주요 Propagation 속성 비교

| 전파 유형          | 트랜잭션 존재 시          | 트랜잭션 없음 시  | 특징                |
| -------------- | ------------------ | ---------- | ----------------- |
| REQUIRED       | 참여                 | 새로 생성      | 가장 일반적인 설정        |
| REQUIRES\_NEW  | 기존 중단 후 새 생성       | 새로 생성      | 독립 트랜잭션 처리에 사용    |
| NESTED         | SavePoint 생성 후 중첩  | 새로 생성      | 부모 롤백 시 전체 롤백     |
| MANDATORY      | 참여                 | 예외 발생      | 상위 트랜잭션이 필요할 때    |
| NEVER          | 예외 발생              | 트랜잭션 없이 실행 | 트랜잭션 없는 작업용       |
| NOT\_SUPPORTED | 트랜잭션 정지 후 비트랜잭션 실행 | 비트랜잭션      | 캐시 등 비트랜잭션 작업에 사용 |
| SUPPORTS       | 참여                 | 트랜잭션 없이 실행 | 읽기 작업 등 유연하게 대응   |

---

## ✅ 3. 내부 구현 개요

Spring은 `TransactionInterceptor` → `TransactionAspectSupport` → `AbstractPlatformTransactionManager` 흐름을 따라 Propagation 전략에 따라 적절한 트랜잭션 동작을 결정합니다.

예:

```java
if (txAttr.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW) {
    suspend(transaction); // 기존 트랜잭션 정지
    startNewTransaction();
}
```

---

## ✅ 4. Kotlin 예제
아래는 7가지 전파 모드별로 **도메인 시나리오 → ASCII flowchart → Kotlin 예제 코드 → 동작 설명 → 추가 코멘트** 순으로 정리한 내용입니다.

---

### 1. REQUIRED (기본값)

**도메인 시나리오**

* **은행 이체**: 고객 A 계좌에서 고객 B 계좌로 금액을 이체

```
[TransferService.transfer]
       |
       +──▶ debit(from)   (참여 ▶ TX1)
       |
       +──▶ credit(to)   (참여 ▶ TX1)
       |
       └── (TX1 커밋)
```

```kotlin
@Service
class TransferService(
    private val acctSvc: AccountService
) {
    @Transactional                                         // propagation = REQUIRED
    fun transfer(from: Long, to: Long, amount: BigDecimal) {
        acctSvc.debit(from, amount)                         // 기존 TX 참여
        acctSvc.credit(to, amount)                         // 기존 TX 참여
    }
}

@Service
class AccountService(private val repo: AccountRepository) {
    @Transactional                                         // propagation = REQUIRED
    fun debit(id: Long, amt: BigDecimal) {                 
        val a = repo.findById(id).orElseThrow()
        a.balance -= amt
        repo.save(a)
    }

    @Transactional
    fun credit(id: Long, amt: BigDecimal) {
        val a = repo.findById(id).orElseThrow()
        a.balance += amt
        repo.save(a)
    }
}
```

* **동작 설명**: `transfer()` 호출 시 트랜잭션(TX1)이 시작되고, `debit`·`credit` 모두 같은 TX1에 참여. 메서드 중간 에러 발생 시 전체 롤백.
* **추가 코멘트**: 가장 흔히 쓰이는 모드. 서비스들이 같은 트랜잭션 경계 내에서 동작하게 함.

---

### 2. REQUIRES\_NEW

**도메인 시나리오**

* **주문 처리 + 감사로그**: 주문 실패 여부와 상관없이 감사 기록은 무조건 별도 커밋

```
[OrderSvc.placeOrder] ──▶ (TX_O 시작)
   ├─▶ audit.logStart  ──▶ (새 TX_A1 시작/커밋)
   ├─▶ biz.place       ──▶ (참여 ▶ TX_O)
   └─▶ audit.logEnd    ──▶ (새 TX_A2 시작/커밋)
   └── (TX_O 커밋)
```

```kotlin
@Service
class OrderService(
    private val biz: BusinessService,
    private val audit: AuditService
) {
    @Transactional
    fun placeOrder(userId: Long, item: String) {
        audit.logStart(userId, item)                       // 독립 TX
        biz.process(userId, item)                          // 메인 TX
        audit.logEnd(userId, item)                         // 독립 TX
    }
}

@Service
class AuditService(private val repo: AuditRepo) {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    fun logStart(u: Long, it: String) {
        repo.save(Audit("START", u, it))
    }
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    fun logEnd(u: Long, it: String) {
        repo.save(Audit("END", u, it))
    }
}
```

* **동작 설명**: `logStart`·`logEnd`는 메인 트랜잭션과 독립된 새 트랜잭션에 각각 커밋. 메인 로직 실패 시에도 감사로그는 남음.
* **추가 코멘트**: 실패해도 반드시 저장해야 할 로그·알림 등에 유용.

---

### 3. NESTED

**도메인 시나리오**

* **재고 검증**: 주문 처리 중 “특정 상품 과다 주문”만 부분 롤백

```
[OrderService.create]
  ├─▶ inventory.reserve     (참여 ▶ TX_P)
  └─▶ fraud.check           (Savepoint S1)
         └── 오류? → rollback to S1
  └── (TX_P 커밋)
```

```kotlin
@Service
class OrderService(
    private val invSvc: InventoryService,
    private val fraud: FraudCheckService
) {
    @Transactional
    fun createOrder(userId: Long, prodId: Long, qty: Int) {
        invSvc.reserve(prodId, qty)                       // 부모 TX
        fraud.validate(prodId, qty)                       // nested TX
    }
}

@Service
class FraudCheckService(private val repo: FraudRepo) {
    @Transactional(propagation = Propagation.NESTED)
    fun validate(prod: Long, qty: Int) {
        if (qty > 100) throw IllegalStateException("과도 주문")
        repo.save(FraudRecord(prod, qty))
    }
}
```

* **동작 설명**: `validate()` 에서 예외 발생 시 해당 중첩 트랜잭션(S1)만 롤백, 부모 트랜잭션은 계속 진행.
* **추가 코멘트**: JDBC Savepoint를 지원하는 데이터소스에서만 동작.

---

### 4. MANDATORY

**도메인 시나리오**

* **리포트 집계**: 배치 스케줄러 안에서만 동작해야 함

```
[BatchJob.run] (no TX) ──▶ report.add  ──▶ 예외!
[BatchJob.run] (@Transactional) ──▶ report.add  ──▶ 정상
```

```kotlin
@Service
class ReportService(private val repo: ReportRepo) {
    @Transactional(propagation = Propagation.MANDATORY)
    fun addReport(data: ReportData) {
        repo.save(DailyReport(data))
    }
}

@Service
class BatchJob(private val rs: ReportService) {
    @Transactional
    fun runJob(data: ReportData) {
        rs.addReport(data)                                 // 상위 TX 필수
    }
}
```

* **동작 설명**: `addReport()` 호출 시 상위 트랜잭션이 없으면 `IllegalTransactionStateException` 발생.
* **추가 코멘트**: 외부에서 직접 호출금지, 반드시 배치 컨텍스트에서만 쓰도록 강제.

---

### 5. NEVER

**도메인 시나리오**

* **시스템 유지보수**: 트랜잭션 중이면 절대 실행되면 안 되는 파일 청소

```
[Maintenance.clean] (TX 있으면 예외)
```

```kotlin
@Service
class MaintenanceService {
    @Transactional(propagation = Propagation.NEVER)
    fun cleanTemp() {
        // 트랜잭션 내부에서 호출 시 예외
        File("/tmp/cache").deleteRecursively()
    }
}
```

* **동작 설명**: 현재 트랜잭션이 감지되면 즉시 예외(`IllegalTransactionStateException`) 발생.
* **추가 코멘트**: 트랜잭션 외부에서만 절대 동작해야 하는 작업에 사용.

---

### 6. NOT\_SUPPORTED

**도메인 시나리오**

* **통계 조회**: 긴 대량 조회 중 트랜잭션 오버헤드 제거

```
[UserSvc.create] ──▶ (TX start)
   └─▶ stats.genReport ──▶ (TX suspend → no TX → 실행 → TX resume)
```

```kotlin
@Service
class StatsService(private val repo: StatsRepo) {
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    fun genReport(): List<Stats> {
        return repo.findAll()                            // 트랜잭션 없이 실행
    }
}

@Service
class UserService(private val stats: StatsService) {
    @Transactional
    fun createUser(u: User) {
        // 사용자 생성 로직...
        stats.genReport()                                // 기존 TX 일시 중단
    }
}
```

* **동작 설명**: `genReport()` 실행 중 기존 트랜잭션 일시 중단(suspend), 완료 후 재개.
* **추가 코멘트**: 읽기 전용이며, 트랜잭션 관리 오버헤드를 없애고 싶을 때.

---

### 7. SUPPORTS

**도메인 시나리오**

* **잔액 조회**: 호출자가 트랜잭션 있으면 참여, 없으면 단순 조회

```
[WebController.view] (no TX) ──▶ balSvc.get  (no TX)
[ServiceMethod]  (TX)      ──▶ balSvc.get  (참여)
```

```kotlin
@Service
class BalanceService(private val repo: AccountRepo) {
    @Transactional(propagation = Propagation.SUPPORTS, readOnly = true)
    fun getBalance(id: Long): BigDecimal {
        return repo.findById(id).orElseThrow().balance
    }
}

@RestController
class AccountController(private val bal: BalanceService) {
    @GetMapping("/balance/{id}")
    fun view(@PathVariable id: Long): BigDecimal {
        return bal.getBalance(id)                       // 트랜잭션 없이 실행
    }
}
```

* **동작 설명**: 이미 트랜잭션이 있으면 그 안에서, 없으면 트랜잭션 없이 실행.
* **추가 코멘트**: 선택적(read-only) 트랜잭션 참여가 필요할 때.

---


## ✅ 5. 요약

| Propagation    | 핵심 용도               |
| -------------- | ------------------- |
| REQUIRED       | 기본, 같은 트랜잭션에서 처리    |
| REQUIRES\_NEW  | 독립 트랜잭션, 실패 격리 필요 시 |
| NESTED         | SavePoint 기반 중첩 처리  |
| MANDATORY      | 반드시 상위 트랜잭션 존재      |
| NEVER          | 트랜잭션 없이 실행 보장       |
| NOT\_SUPPORTED | 트랜잭션 중단 후 비트랜잭션 실행  |
| SUPPORTS       | 있으면 참여, 없으면 비트랜잭션   |

---

## 내 생각
> 다양한 도메인 환경에서 트랜잭션 전파 설정을 할수 있다. 현대 아키텍트에서는 주로 트랜잭션 레벨로 도메인을 분리하거나 서비스를 분리하기 때문에 어쩌면 디테일한 내용을 모르고 개발하는 경우도 많을 것 같다.
> 
> 마이크로 서비스가 매우 좋은 아키텍처이긴 하나 개발과 운영상 허들과 높은 수준의 노하우를 필요로 한다. 
> 따라서 경우에 따라 모노리스 환경에서 개발이 빈번하게 시도 될수도 있다. 모노리스 개발은 다양한 도메인과 여러 디비가 사용될 수 있으므로 해당 지식을 매우 필수적일 것 같다.   
> 현재 사내에서 개발중인 플랫폼 개발도 DDD 기반의 모노리스 형태로 개발되며 트랜잭션간 상호 작용과 전파에 대해 매우 민감하게 처리되고 있다. 
> 
> 이에 따라 도메인 처리에 최적화된 트랜잭션 설계도 필수라는 생각이 든다.