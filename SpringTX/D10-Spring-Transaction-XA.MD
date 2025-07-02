# **개념 및 배경**

## **1. 다중 데이터소스(Transaction per DataSource) vs. 분산 트랜잭션**

### * **다중 데이터소스**:

    * 애플리케이션에서 두 개 이상의 `DataSource`(예: 주문 DB, 재고 DB)를 사용.
    * 각 `DataSource`에 대해 독립적인 로컬 트랜잭션(`DataSourceTransactionManager`)을 사용하면, 트랜잭션 경계를 각각 관리하게 됨.
    * 예: 주문 저장은 A 트랜잭션, 재고 차감은 B 트랜잭션.

### * **분산 트랜잭션(XA/2PC)**:

    * 여러 자원(Resource Manager)에 걸친 하나의 논리적 트랜잭션을 보장.
    * **2단계 커밋(2PC, Two-Phase Commit)** 프로토콜 사용.

        1. **Prepare(준비)**: 모든 참여 자원에 “커밋 가능?”을 요청하고, 성공 응답을 받으면
        2. **Commit(확정)**: 실제 커밋 실행.
    * 실패 시 전부 롤백.

---

### **ChainedTransactionManager**

ChainedTransactionManager는 여러 개의 로컬 PlatformTransactionManager를 묶어 한 트랜잭션처럼 시작하고, 커밋·롤백을 순차적으로 처리해 주는 Spring Data Commons의 구현체입니다.

각기 다른 데이터소스에 대해 개별 트랜잭션 매니저(DataSourceTransactionManager 등)를 지정하고, 이를 “체인(chain)” 구조로 묶어 사용

진정한 2단계 커밋(XA)은 아니지만, 간편하게 여러 로컬 트랜잭션을 함께 관리할 때 유용

  ```mermaid
  flowchart LR
    A([Begin]) --> B["tm1.getTransaction()"]
    B --> C["tm2.getTransaction()"]
    C --> D{비즈니스 로직 수행}
    D --> E["tm2.commit()"]
    E --> F["tm1.commit()"]
    D -- "예외 발생" --> G["tm2.rollback()"]
    G --> H["tm1.rollback()"]
  ```

### 클래스 위치 및 주요 메서드
```java
package org.springframework.data.transaction;

public class ChainedTransactionManager implements PlatformTransactionManager {
  public ChainedTransactionManager(PlatformTransactionManager... transactionManagers) { … }

  @Override
  public TransactionStatus getTransaction(TransactionDefinition definition) { … }

  @Override
  public void commit(TransactionStatus status) throws TransactionException { … }

  @Override
  public void rollback(TransactionStatus status) throws TransactionException { … }
}

```
- 생성자: 관리할 PlatformTransactionManager 리스트를 순서대로 받음 (null·empty 금지)
- getTransaction(): 내부의 각 TM을 입력 순서대로 호출해 TransactionStatus를 누적
- commit(): 커밋은 역순(reverse order) 으로 수행
- rollback(): 롤백 역시 역순으로 수행


### 커밋·롤백 시퀀스
- 시작: 등록된 TM을 선언한 순서대로 getTransaction() 호출
- 커밋: 내부 리스트의 역순으로 commit(status) 실행
- 중간에 커밋 실패 시 HeuristicCompletionException 발생 → 부분 커밋 상태를 알림
- 롤백: 역시 역순으로 rollback(status) 실행
- 커밋 실패가 없더라도, 필요 시 각 TM에 롤백 호출

### 예외 처리 및 한계
- 부분 커밋 위험
    - 첫 번째 TM 커밋 성공 후, 두 번째 TM 커밋에서 I/O 오류 등으로 실패하면 → 첫 TM은 그대로 커밋된 상태
    - 이 때 commit(...)은 HeuristicCompletionException을 던져 부분 커밋을 알림
- 롤백 보장 한계
    - 커밋 전에 발생한 예외는 모든 TM에 정상 롤백 호출
    - 커밋 단계에서의 예외는 이미 커밋된 리소스에 대해 롤백할 수 없음 → 데이터 불일치 가능
- 따라서 “부분 커밋 상태를 애플리케이션이 용인하거나, 이후 보상 처리(compensation)를 설계”해야 함


### 왜 deprecated 되었나?
- 부분 커밋(Heuristic) 문제
    - 내부적으로 여러 AbstractPlatformTransactionManager를 순차적으로 호출하며, commit/rollback 시 reverse order로 처리합니다.
    - 하지만 “첫 번째 TM이 commit” → “두 번째 TM에서 예외” 발생 시, 이미 모든 TransactionSynchronization이 첫 번째 TM에 의해 처리되어 버려 롤백이나 보상이 불가능합니다.
    - 이러한 동기화 저장소(synchronization storage)가 싱글턴 ThreadLocal에 의존하는 구조적 한계 때문에, 완벽한 트랜잭션 일관성을 보장할 수 없습니다.

- 예측하지 못한 롤백 동작
    - 2단계 커밋이 아니므로, 중간 단계 실패 시 복구 로직을 프레임워크가 제공하지 않습니다.
- 결과적으로 “애플리케이션이 부분 커밋 상태를 직접 허용하거나 보상(compensation)을 구현”해야만 사용이 가능한 패턴이라, 일반적인 사용에는 부적합하다고 판단되었습니다.

### 대안 TransactionSynchronization 등록
- 단순히 “커밋 전/후 콜백”이 목적이라면, ChainedTransactionManager 대신 Spring의 TransactionSynchronization 을 사용하세요.

- 예시:

```java
TransactionSynchronizationManager.registerSynchronization(
  new TransactionSynchronizationAdapter() {
    @Override
    public void beforeCommit(boolean readOnly) { … }
    @Override
    public void afterCommit() { … }
  }
);
```
- 트랜잭션 라이프사이클에 맞춰 리소스를 안전하게 정리하거나 후처리할 수 있습니다.

---

### **XA 트랜잭션 기초**

* **X/Open XA**: 분산 트랜잭션을 위한 표준 인터페이스.
* **JTA(Java Transaction API)**: 자바 환경에서 XA를 사용할 수 있도록 정의된 API.
* **JtaTransactionManager**: Spring이 JTA 구현체(e.g. Atomikos, Bitronix, Narayana)를 래핑한 TM.
* **흐름 (2PC)**:

  ```mermaid
  flowchart LR
    A(Start TX) --> B["TM.begin()"]
    B --> C["RM1.prepare()"]
    C --> D["RM2.prepare()"]
    D --> E{모두 준비 완료?}
    E -- 예 --> F["RM1.commit()"]
    F --> G["RM2.commit()"]
    E -- 아니오 --> H["RM1.rollback()"]
    H --> I["RM2.rollback()"]
  ```
* **구성 요소**:

    * **TransactionManager**: 트랜잭션 라이프사이클 관리
    * **XAResource**: 각 DataSource(또는 JMS, JCA 등)가 제공
    * **TransactionLog**: 트랜잭션 상태 영속화

---

## 예제 코드 (Kotlin)

**도메인**: 고객 주문(`Order`) 저장 후, 재고 차감(`Inventory`)

### 1) As-Is: 로컬 트랜잭션을 분리해 사용

```kotlin
@Service
class OrderService(
    private val orderRepo: OrderRepository,
    private val inventoryRepo: InventoryRepository
) {
    // 주문 DB 전용 트랜잭션
    @Transactional("orderTxManager")
    fun saveOrder(order: Order) {
        orderRepo.save(order)
        reduceStock(order)
    }

    // 재고 DB 전용 트랜잭션
    @Transactional("inventoryTxManager")
    fun reduceStock(order: Order) {
        inventoryRepo.decrease(order.productId, order.quantity)
    }
}
```

* **문제**: `saveOrder` → `reduceStock` 호출 시, 두 트랜잭션 간 커밋 시점이 독립적이어서 중간 실패 시 데이터 불일치 가능.

---

### 2) To-Be A: ChainedTransactionManager 도입

```kotlin
@SpringBootApplication
class TransactionConfig {
    @Bean
    fun chainedTxManager(
        orderTxManager: PlatformTransactionManager,
        inventoryTxManager: PlatformTransactionManager
    ): PlatformTransactionManager =
        ChainedTransactionManager(orderTxManager, inventoryTxManager)
}

@Service
class OrderServiceV2(
    private val orderRepo: OrderRepository,
    private val inventoryRepo: InventoryRepository
) {
    // 묶인 트랜잭션 매니저 사용
    @Transactional("chainedTxManager")
    fun placeOrder(order: Order) {
        orderRepo.save(order)
        inventoryRepo.decrease(order.productId, order.quantity)
    }
}
```

* **특징**: 두 로컬 TM을 순차(commit/rollback)로 실행
* **주의**: 진정한 2PC가 아니므로, 중간 오류 시 완벽한 롤백은 보장되지 않음.

---

### 3) To-Be B: XA (JTA) 트랜잭션 적용

```kotlin
@SpringBootApplication
class JtaConfig {
    // Atomikos 같은 JTA 구현체 의존 후
    @Bean
    fun transactionManager(): JtaTransactionManager =
        JtaTransactionManager()
}

@Service
class OrderServiceV3(
    private val orderRepo: OrderRepository,
    private val inventoryRepo: InventoryRepository
) {
    @Transactional  // 기본 JTA TM이 선택됨
    fun placeOrder(order: Order) {
        orderRepo.save(order)
        inventoryRepo.decrease(order.productId, order.quantity)
    }
}
```

* **장점**: 두 DB 모두 XAResource로 등록되어 **2PC** 로직에 따라 전 구간 ACID 보장
* **단점**: 설정 복잡도 증가, XA 드라이버/미들웨어 비용

---

## 요약

1. **다중 데이터소스** → 로컬 TM 여러 개를 사용하는 것.
2. **ChainedTransactionManager** → 로컬 TM을 순차 묶음–베스트 에포트 방식.
3. **XA 트랜잭션(JTA)** → 진정한 2단계 커밋으로 분산 트랜잭션 ACID 보장.
4. **선택 포인트**

    * **간단 통합** → ChainedTM
    * **강력 ACID** → JTA(XA) + JtaTransactionManager

이제 실제 환경과 요구사항(정합성, 운영 복잡도, 성능 등)을 고려해 적절한 방식을 선택하면 됩니다.
