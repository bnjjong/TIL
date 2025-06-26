## 1. 롤백(Rollback)의 정의와 개념

* **롤백(Rollback)**
  트랜잭션 내에서 발생한 모든 변경을 취소하고, 트랜잭션 시작 전 상태로 복원하는 것.
  이는 ACID 특성 중 **원자성(Atomicity)** 보장을 위해 필요하다.

* **예외 발생 시 트랜잭션 처리**

    * **성공 시**: 커밋 → 모든 변경이 영구 반영
    * **실패(예외) 시**: 롤백 → 변경사항 전부 취소

---

## 2. Spring의 기본 롤백 규칙

1. **언체크드 예외(런타임 예외) 및 Error**
   기본적으로 `RuntimeException` 및 그 하위 예외, `Error` 계열 예외가 발생하면 자동으로 롤백
2. **체크드 예외(일반 Exception)**
   `Exception` 계열(단, `RuntimeException` 제외) 발생 시 기본적으로 커밋
3. **커스텀 설정**

    * `@Transactional(rollbackFor = [CustomCheckedException::class])`
    * `@Transactional(noRollbackFor = [SomeRuntimeException::class])`
      을 통해 롤백 대상을 세밀하게 지정 가능

---

## 3. 예외 처리 방식과 배경 지식

### 3.1 선언적 트랜잭션 vs 프로그래매틱 트랜잭션

* **선언적(@Transactional)**
  AOP 프록시를 이용해 메서드 진입 전후로 트랜잭션 시작/커밋·롤백
* **프로그래매틱(PlatformTransactionManager 직접 사용)**
  `TransactionStatus` 객체를 직접 받아 `commit()`/`rollback()` 호출

### 3.2 내부 구현 핵심 클래스

* `TransactionInterceptor` → 메서드 호출 전후로 트랜잭션 제어
* `TransactionAspectSupport` → `TransactionAttribute` 파싱, 롤백 규칙 관리
* `PlatformTransactionManager` → 실제 커밋/롤백 구현 (`DataSourceTransactionManager`, `JpaTransactionManager` 등)
* `RollbackRuleAttribute` → 어떤 예외에 롤백할지 판단하는 룰 객체

---

## 4. Kotlin 예제 코드 (도메인 기반)

### 도메인 설명

* **OrderService**: 주문을 생성하고, 재고를 차감한다.
* **BusinessCheckedException**: 비즈니스 검증 실패 시 던지는 체크드 예외
* **RuntimeBusinessException**: 런타임 예외로 정의된 비즈니스 예외

---

### 4.1 As-Is 코드

```kotlin
@Service
class OrderService(
    private val orderRepository: OrderRepository,
    private val inventoryService: InventoryService
) {

    // 기본 설정: 체크드 예외 발생 시 커밋이 되버린다!
    @Transactional
    fun placeOrder(orderDto: OrderDto) {
        // 1) 주문 저장
        val order = orderRepository.save(Order.from(orderDto))

        // 2) 재고 차감 중 비즈니스 체크 예외 발생
        try {
            inventoryService.decreaseStock(orderDto.productId, orderDto.quantity)
        } catch (ex: BusinessCheckedException) {
            // 단순 로깅만 하고 예외를 swallow → 트랜잭션 커밋
            log.error("재고 부족: ${ex.message}")
        }

        // 3) 추가 작업...
    }
}
```

* **문제점**

    * `BusinessCheckedException`이 발생해도 롤백되지 않고 커밋
    * catch 후 재예외를 던지지 않으므로 트랜잭션 종료 시 커밋

---

### 4.2 To-Be 코드

```kotlin
@Service
class OrderService(
    private val orderRepository: OrderRepository,
    private val inventoryService: InventoryService
) {

    // 체크드 예외도 롤백 대상으로 포함
    @Transactional(rollbackFor = [BusinessCheckedException::class])
    fun placeOrder(orderDto: OrderDto) {
        // 1) 주문 저장
        val order = orderRepository.save(Order.from(orderDto))

        // 2) 재고 차감. 부족 시 체크드 예외 던지면 롤백
        inventoryService.decreaseStock(orderDto.productId, orderDto.quantity)

        // 3) 추가 작업...
    }
}
```

```kotlin
@Service
class InventoryService(
    private val inventoryRepository: InventoryRepository
) {
    fun decreaseStock(productId: Long, qty: Int) {
        val item = inventoryRepository.findById(productId)
            .orElseThrow { RuntimeException("상품을 찾을 수 없습니다.") }

        if (item.stock < qty) {
            // 체크드 예외 던지기
            throw BusinessCheckedException("남은 수량(${item.stock})이 부족합니다.")
        }
        item.stock -= qty
        inventoryRepository.save(item)
    }
}

// 사용자 정의 체크드 예외
class BusinessCheckedException(message: String): Exception(message)
```

* **개선 포인트**

    1. `rollbackFor`로 체크드 예외도 롤백 대상에 추가
    2. 예외를 잡아서 swallow 하지 말고, 프레임워크에 위임하여 롤백 처리

---

## 5. 요약

* **롤백 기본 룰**: 런타임 예외(`RuntimeException`), `Error` → 롤백 / 체크드 예외(`Exception`) → 커밋
* **커스터마이징**: `rollbackFor`, `noRollbackFor` 속성으로 세밀하게 제어 가능
* **선언적 트랜잭션**: AOP 기반으로 `@Transactional` 사용 → 내부적으로 `TransactionInterceptor`가 처리
* **예제 핵심**:

    * As-Is: 체크드 예외를 catch 후 swallow → 커밋
    * To-Be: `rollbackFor` 지정 + 예외를 던져 프레임워크에 롤백 위임

이로써 “롤백 규칙과 예외 처리”에 대한 이론과 실제 코드 적용 사례를 함께 살펴보았습니다. 다음 스터디에서도 도메인에 녹아든 구체적 예제를 통해 이해를 더 심화해 보시면 좋겠습니다.
