## 1. 트랜잭션 정의 및 개념

### 트랜잭션(Transaction)이란

* **하나의 논리적 작업 단위**로, 여러 DB 연산을 묶어서 “모두 성공”하거나 “모두 실패”시켜 데이터 일관성을 보장하는 메커니즘
* 트랜잭션이 시작되면, 커밋 전까지 수행된 변경은 외부에 노출되지 않고, 예외 발생 시 롤백되어 이전 상태로 복구됨

### ACID 특성

1. **Atomicity(원자성)**

    * 트랜잭션 내 연산은 하나의 단위로 실행. 일부만 실행되고 실패하면 전부 롤백
2. **Consistency(일관성)**

    * 트랜잭션 전·후에 DB 제약(무결성, 외래키, 도메인 규칙 등)이 항상 유지
3. **Isolation(독립성)**

    * 동시 실행되는 트랜잭션끼리 간섭 금지. 중간 결과가 다른 트랜잭션에 보이지 않음
4. **Durability(지속성)**

    * 커밋된 결과는 시스템 장애 후에도 영구 보존

---

## 2. 배경 지식

### 2.1 왜 트랜잭션이 필요한가?

* **데이터 불일치 방지**

    * 예: 재고 차감 후 주문 생성 중 결제 오류 발생 → 재고만 빠지고 주문이 없으면 일관성 깨짐
* **업무 로직의 원자성 보장**

    * 여러 처리(재고·주문·포인트 적립·알림 발송 등)가 하나의 단위로 성공·실패

### 2.2 DB 트랜잭션 vs Spring 트랜잭션

* **DB 트랜잭션**

    * JDBC, JTA 등을 직접 사용해 `connection.commit()`, `connection.rollback()` 제어
* **Spring 선언적 트랜잭션**

    * `PlatformTransactionManager` 가 구현별(JDBC, JPA, JTA)로 동작을 추상화
    * `@Transactional` + AOP 프록시로 애플리케이션 코드에 트랜잭션 경계 선언
    * 비즈니스 로직에만 집중하고, 설정만으로 트랜잭션 관리

---

## 3. 예제 코드 (AS-IS → TO-BE)

### 도메인: 전자상거래 주문 처리

* **요구사항**

    1. 주문 수량만큼 상품 재고 차감
    2. 주문 정보 생성
    3. 결제 한도(5개 초과) 초과 시 전체 롤백

---

### 3.1 AS-IS (트랜잭션 미적용)

```kotlin
@Service
class OrderService(
    private val productRepo: ProductRepository,
    private val orderRepo: OrderRepository
) {
    fun placeOrder(userId: Long, productId: Long, quantity: Int) {
        // 1) 재고 차감
        val product = productRepo.findById(productId)
            .orElseThrow { IllegalArgumentException("상품 없음: $productId") }
        if (product.stock < quantity)
            throw IllegalStateException("재고 부족: 현재 ${product.stock}, 요청 $quantity")
        product.stock -= quantity
        productRepo.save(product)

        // 2) 주문 생성
        val order = Order(userId = userId, productId = productId, quantity = quantity)
        orderRepo.save(order)

        // 3) 결제 한도 체크 중 예외 발생
        if (quantity > 5)
            throw IllegalStateException("결제 한도 초과: $quantity 개")
    }
}
```

> **문제점**
>
> * `quantity > 5` 예외 발생 시, 이미 커밋된 재고 차감은 롤백되지 않아 주문과 재고가 불일치

---

### 3.2 TO-BE (트랜잭션 적용)

```kotlin
@Service
class OrderService(
    private val productRepo: ProductRepository,
    private val orderRepo: OrderRepository
) {
    @Transactional
    fun placeOrder(userId: Long, productId: Long, quantity: Int) {
        // 1) 재고 차감
        val product = productRepo.findById(productId)
            .orElseThrow { IllegalArgumentException("상품 없음: $productId") }
        if (product.stock < quantity)
            throw IllegalStateException("재고 부족: 현재 ${product.stock}, 요청 $quantity")
        product.stock -= quantity
        productRepo.save(product)

        // 2) 주문 생성
        val order = Order(userId = userId, productId = productId, quantity = quantity)
        orderRepo.save(order)

        // 3) 결제 한도 체크
        if (quantity > 5)
            throw IllegalStateException("결제 한도 초과: $quantity 개")
    }
}
```

* `@Transactional` 선언으로 **메서드 전체가 하나의 DB 트랜잭션**으로 묶임
* 내부 예외 발생 시 **전부 롤백** → 재고 차감, 주문 생성 모두 취소되어 데이터 일관성 보장

---

## 4. 요약

* **트랜잭션**: 여러 DB 연산을 하나의 논리 단위로 묶어 “전부 성공 or 전부 실패” 보장
* **ACID**: Atomicity, Consistency, Isolation, Durability
* **Spring 트랜잭션**: `@Transactional` + AOP 프록시 기반 선언적 관리
* **핵심**: 비즈니스 로직에 트랜잭션 경계만 선언하면, 에러 시 자동 롤백으로 데이터 무결성 유지 가능
