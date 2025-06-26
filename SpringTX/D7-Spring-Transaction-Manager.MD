## **1. 프로그래밍적 트랜잭션 관리란?**

Spring에서는 트랜잭션 관리를 크게 두 가지 방식으로 제공합니다.

* **선언적(Declarative) 트랜잭션 관리**: `@Transactional` 애노테이션을 통해 AOP 기반으로 트랜잭션 범위를 지정
* **프로그래밍적(Programmatic) 트랜잭션 관리**: 코드 안에서 직접 `PlatformTransactionManager`나 `TransactionTemplate`을 이용하여 트랜잭션 시작·커밋·롤백을 제어

프로그래밍적 트랜잭션 관리는

* 코드 흐름 상에서 트랜잭션 경계를 명확히 제어해야 할 때
* 반복적인 트랜잭션 설정(격리 수준, 타임아웃 등)을 캡슐화하고 재사용하고 싶을 때
* 혹은 선언적 방식이 적용되지 않는 객체(예: 스프링 빈이 아닌 일반 자바 객체)에서 트랜잭션을 처리해야 할 때 유용합니다.

---

## **2. `TransactionTemplate` 사용법**

* `TransactionTemplate`은 내부적으로 `PlatformTransactionManager`와 `TransactionDefinition`(기본은 `DefaultTransactionDefinition`)을 조합하여, `execute(...)` 호출 시 트랜잭션을 시작하고, 콜백 실행 후 자동으로 커밋 혹은 예외 발생 시 롤백을 수행합니다.
* 주로 빈 설정에서 `PlatformTransactionManager`를 주입받아 간단히 생성해서 사용합니다.

```kotlin
// 빈 설정 예시 (Spring Boot @Configuration)
@Configuration
class TransactionConfig(
    private val txManager: PlatformTransactionManager
) {
    @Bean
    fun transactionTemplate(): TransactionTemplate =
        TransactionTemplate(txManager).apply {
            // 기본 격리 수준, 타임아웃 등을 여기서 설정 가능
            // isolationLevel = TransactionDefinition.ISOLATION_READ_COMMITTED
            // timeout = 30
        }
}
```

* 이후 필요한 서비스 클래스에서 `TransactionTemplate`을 주입받아 사용합니다.

```kotlin
@Service
class MyService(
    private val txTemplate: TransactionTemplate,
    private val orderRepository: OrderRepository
) {
    fun processOrder(order: Order) {
        txTemplate.execute { status ->
            // 트랜잭션 경계 내에서 실행할 코드
            orderRepository.save(order)
            // 예외 발생 시 status.setRollbackOnly() 로 롤백 지정 가능
        }
    }
}
```

---

## **3. `TransactionCallback` 예제**

* `execute` 메서드는 `TransactionCallback<T>` 인터페이스를 인자로 받으며, 내부적으로 `doInTransaction(txStatus)` 메서드를 호출합니다.
* 람다를 사용하면 `TransactionCallback<T>`를 간결하게 표현할 수 있습니다.

```kotlin
txTemplate.execute<Boolean> { status ->
    try {
        // 비즈니스 로직
        performBusinessLogic()
        true  // 호출자에게 결과 반환
    } catch (ex: Exception) {
        status.setRollbackOnly()  // 명시적 롤백
        false
    }
}
```

---

## **4. 내부 구현(간략)**

* `TransactionTemplate.execute(callback)` 호출 시:

    1. `txManager.getTransaction(definition)` → `TransactionStatus` 반환
    2. `callback.doInTransaction(status)` 실행
    3. 정상 종료 시 `txManager.commit(status)`, 예외 시 `txManager.rollback(status)`

이 로직은 `TransactionTemplate` 클래스 내부의 `execute` 메서드에 구현되어 있습니다.

---

## **5. 예제 코드: 비즈니스 도메인(주문 처리) – As-Is vs To-Be**

### 5.1 As-Is: 직접 `PlatformTransactionManager` 사용

```kotlin
@Service
class OrderServiceAsIs(
    private val txManager: PlatformTransactionManager,
    private val orderRepository: OrderRepository,
    private val paymentGateway: PaymentGateway
) {
    fun placeOrder(order: Order): Receipt {
        // 1. 트랜잭션 정의
        val def = DefaultTransactionDefinition().apply {
            isolationLevel = TransactionDefinition.ISOLATION_READ_COMMITTED
            timeout = 30
        }
        // 2. 트랜잭션 시작
        val status = txManager.getTransaction(def)
        return try {
            // 비즈니스 로직
            orderRepository.save(order)
            val receipt = paymentGateway.charge(order)
            // 커밋
            txManager.commit(status)
            receipt
        } catch (ex: Exception) {
            // 롤백
            txManager.rollback(status)
            throw ex
        }
    }
}
```

### 5.2 To-Be: `TransactionTemplate` 적용

```kotlin
@Service
class OrderServiceToBe(
    private val txTemplate: TransactionTemplate,
    private val orderRepository: OrderRepository,
    private val paymentGateway: PaymentGateway
) {
    fun placeOrder(order: Order): Receipt =
        txTemplate.execute<Receipt> { status ->
            // 1. 주문 저장
            orderRepository.save(order)
            // 2. 결제 처리
            val receipt = paymentGateway.charge(order)
            // 결과 반환하면 자동 커밋
            receipt
        } ?: throw IllegalStateException("트랜잭션 실행 중 문제가 발생했습니다.")
}
```

## * **차이점 요약**

    * As-Is: `getTransaction` → try/catch → `commit`/`rollback` 직접 호출
    * To-Be: `execute` 람다 내부에 비즈니스 로직만 작성, 커밋·롤백은 템플릿이 자동 처리

---

## **6. 요약**

* **프로그래밍적 트랜잭션 관리**는 코드 안에서 직접 트랜잭션 경계를 제어해야 할 때 사용.
* \*\*`TransactionTemplate`\*\*은 `PlatformTransactionManager`와 트랜잭션 설정을 래핑하여, `execute(callback)`만으로 커밋·롤백을 자동화해줌.
* \*\*`TransactionCallback`\*\*는 트랜잭션 내에서 실행할 로직을 정의하는 인터페이스로, 람다로 간결히 표현 가능.
* As-Is(직접 호출) 방식보다 To-Be(`TransactionTemplate`) 방식이 코드 가독성·재사용성·유지보수성에서 우수.


## 내 생각
> 트랜잭션 매니저의 경우 JPA 설정시 Bean으로 등록하고 선언적 트랜잭션으로 주로 활용했었다. 
> 사실 대부분의 비지니스에서는 특별히 프로그래밍적 트랜잭션을 사용할일이 없었던 것 같다.
> 
> 비지니스가 심플하고 도메인별로 서비스가 잘 분리 되어 있다면 사용할 일이 많지는 않을 것 같다.
> 다만 어떤 케이스에서 사용해야 될지는 고려해두는 것이 좋을 것 같다.
> 
> 최근에 대용량 업로드 배치에서 프로그래밍적 트랜잭션을 사용한 케이스는 있다. 확실히 대용량 처리나 트랜잭션처리의 커스텀이 필요하다면 필요한 순간이 있을 것 같다.
> 