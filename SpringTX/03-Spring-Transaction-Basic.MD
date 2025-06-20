# 1. **@Transactional 기본 사용법 – 정의 및 개념**

## **어노테이션 위치(클래스 vs 메서드)**
- `클래스 레벨`에 선언하면 그 클래스의 `모든 public 메서드`에 트랜잭션이 적용된다.
- 메서드 레벨에 선언하면 해당 메서드에만 적용되며, `클래스 레벨 설정을 덮어쓸` 수 있다.
- 우선순위: 메서드 > 클래스. 세밀한 제어가 필요하면 메서드 위에 직접 붙이자.

## **readOnly 옵션**
- `@Transactional(readOnly = true)`를 사용하면 트랜잭션 매니저에 `"이 트랜잭션에서 데이터 변경은 없다"`는 힌트를 준다.
- JPA의 경우 flush(변경 감지)를 생략해 성능 최적화, DB 락 경합 완화
- 읽기 전용 조회 메서드(검색, 통계 등)에 일관되게 적용하면 좋다.

## **timeout 설정**
- `@Transactional(timeout = N)`으로 트랜잭션 최대 지속 시간을 초 단위로 지정
- 시간이 초과되면 `TransactionTimedOutException`이 발생하며 자동 롤백
- 장기 실행 로직(대량 데이터 처리, 외부 API 호출 등)에 보호막을 씌운다고 생각하자.

# **배경 지식**

## **왜 위치 제어가 필요할까?**
- 트랜잭션 오버헤드는 무시할 수 없다. 꼭 필요한 코드 블록에만 적용해야 성능과 가독성이 모두 좋아진다.
## **readOnly 힌트의 내부 작동**
-  DataSourceTransactionManager나 JpaTransactionManager가 내부에서 커넥션/세션을 `readOnly` 모드로 설정하거나, JPA flush를 생략
## **timeout이 동작하는 원리**
- Spring의 `DefaultTransactionDefinition`에 설정된 timeout 값이 `PlatformTransactionManager.getTransaction()` 시 인자로 전달되고, `내부 타이머 검사`를 통해 초과 시 예외 트리거

<details>
    <summary>코드 보기</summary>

```java
public class JtaTransactionManager extends AbstractPlatformTransactionManager {

    @Override
    protected void doBegin(Object transaction, TransactionDefinition definition) {
        try {
            UserTransaction ut = getUserTransaction();  // JNDI lookup 등으로 획득
            int timeout = definition.getTimeout();
            if (timeout > TransactionDefinition.TIMEOUT_DEFAULT) {
                // 여기서 JTA 레벨 타임아웃 설정
                ut.setTransactionTimeout(timeout);
            }
            ut.begin();  // 타임아웃 카운트다운 시작
            // 이후 TransactionManager.getTransaction() 으로 트랜잭션 참가 처리
        }
        catch (Exception ex) {
            throw new CannotCreateTransactionException("Could not begin JTA transaction", ex);
        }
    }

    @Override
    protected void doCommit(DefaultTransactionStatus status) {
        try {
            getUserTransaction().commit();  // commit 시도
        }
        catch (Exception ex) {
            throw new TransactionSystemException("Could not commit JTA transaction", ex);
        }
    }

    @Override
    protected void doRollback(DefaultTransactionStatus status) {
        try {
            getUserTransaction().rollback();
        }
        catch (Exception ex) {
            throw new TransactionSystemException("Could not rollback JTA transaction", ex);
        }
    }
}
```

</details>


# 3. **내구성(Durability) 구현 코드 예시**

- 트랜잭션 커밋 시 Spring이 실제로 `Connection.commit()`을 호출해 DB 로그(undo/redo)에 기록을 남겨 영구 보존을 보장한다.

   ```java
   // Spring Framework: DataSourceTransactionManager#doCommit()
   @Override
   protected void doCommit(DefaultTransactionStatus status) {
     ConnectionHolder conHolder = (ConnectionHolder) status.getTransaction();
     try {
       conHolder.getConnection().commit();   // <-- 여기서 DB 엔진에 커밋 명령, 로그 파일에 기록
     } catch (SQLException ex) {
       throw new CannotCommitException("Could not commit JDBC transaction", ex);
     }
   }
   ```

# 4. **예제 코드 (AS-IS → TO-BE)**
   *도메인: 간단한 결제 처리 서비스 (PaymentService)*

    * 요구사항

        1. 사용자 계정에서 잔액 차감
        2. 결제 내역 기록
        3. 기록 단계에서 오류 발생 시 전체 롤백
        4. 조회 메서드엔 읽기전용, 기록 메서드엔 3초 타임아웃

   ## **4.1 AS-IS (트랜잭션 미적용)**

   ```kotlin
   @Service
   class PaymentService(
     private val accountRepo: AccountRepository,
     private val paymentLogRepo: PaymentLogRepository
   ) {
     fun getBalance(userId: Long): BigDecimal {
       val acct = accountRepo.findById(userId)
         .orElseThrow { IllegalArgumentException("사용자 없음: $userId") }
       return acct.balance
     }

     fun processPayment(userId: Long, amount: BigDecimal) {
       // 1) 잔액 차감
       val acct = accountRepo.findById(userId).orElseThrow()
       if (acct.balance < amount) throw IllegalStateException("잔액 부족")
       acct.balance -= amount
       accountRepo.save(acct)

       // 2) 결제 로그 기록
       paymentLogRepo.save(PaymentLog(userId = userId, amount = amount, timestamp = LocalDateTime.now()))

       // 3) 에러 시 재고 차감은 롤백되지 않아 문제 발생
       if (amount > BigDecimal(1000)) throw IllegalStateException("결제 한도 초과")
     }
   }
   ```

   *문제점: `amount > 1000` 예외 발생 시 잔액 차감만 반영되고, 로그 기록 전까지 커밋되어 데이터 불일치*

  ## **4.2 TO-BE (@Transactional 적용)**

   ```kotlin
   @Service
   class PaymentService(
     private val accountRepo: AccountRepository,
     private val paymentLogRepo: PaymentLogRepository
   ) {
     @Transactional(readOnly = true)
     fun getBalance(userId: Long): BigDecimal {
       val acct = accountRepo.findById(userId)
         .orElseThrow { IllegalArgumentException("사용자 없음: $userId") }
       return acct.balance
     }

     @Transactional(timeout = 3)
     fun processPayment(userId: Long, amount: BigDecimal) {
       // 1) 잔액 조회 및 차감
       val acct = accountRepo.findById(userId).orElseThrow()
       if (acct.balance < amount) throw IllegalStateException("잔액 부족")
       accountRepo.save(acct.copy(balance = acct.balance - amount))

       // 2) 결제 로그 기록
       paymentLogRepo.save(PaymentLog(userId = userId, amount = amount, timestamp = LocalDateTime.now()))

       // 3) 결제 한도 초과 시 3초 안에 자동 롤백
       if (amount > BigDecimal(1000)) throw IllegalStateException("결제 한도 초과")
     }
   }
   ```

- `getBalance`는 읽기 전용 트랜잭션 → JPA flush 생략, 락 최소화
- `processPayment`는 3초 timeout 설정 → 장시간 지연 시 `TransactionTimedOutException` 발생하며 전체 롤백

# 5. **요약**

- **어노테이션 위치**: 클래스 vs 메서드, 우선순위는 메서드 > 클래스
- **readOnly**: 조회 전용 트랜잭션 힌트, JPA flush 생략으로 성능·락 경합 완화
- **timeout**: 트랜잭션 최대 지속 시간 지정, 시간 초과 시 자동 롤백으로 장기 실행 보호
- **내구성**: `PlatformTransactionManager.commit()` → `Connection.commit()` 호출로 DB 로그에 기록, 커밋 후에도 데이터 영구 보존
  - Connection.commit 시
    - 디스크 로그에 기록 → 내구성 확보
    - 락 해제 및 리소스 반환 → 동시성 성능 유지
    - 제약 조건 검사 → 일관성 보장
- **실전 예제**: 결제 서비스에 선언적 트랜잭션 적용으로 중간 예외 시 전체 롤백 및 옵션별 최적화 달성

## PlatformTransactionManager commit 코드 참조


```java
public void commit(TransactionStatus status) throws TransactionException {
    DefaultTransactionStatus defStatus = (DefaultTransactionStatus) status;

    // 1) 이미 롤백 마크가 있으면 예외
    if (defStatus.isCompleted()) {
        throw new IllegalTransactionStateException("Transaction is already completed - do not call commit or rollback more than once per transaction");
    }
    if (defStatus.isLocalRollbackOnly()) {
        // 롤백 전환: 롤백 전용 상태가 설정돼 있으면 commit 대신 rollback 처리
        processRollback(defStatus);
        return;
    }

    try {
        // 2) 트랜잭션 동기화 - beforeCommit 콜백
        triggerBeforeCommit(defStatus);
        // 3) 트랜잭션 동기화 - beforeCompletion 콜백
        triggerBeforeCompletion(defStatus);

        // 4) 실제 커밋 작업 수행 (구현체에 위임)
        doCommit(defStatus);

        // 5) 트랜잭션 동기화 - afterCommit 콜백
        triggerAfterCommit(defStatus);
        // 6) 트랜잭션 동기화 - afterCompletion(Comitted) 콜백
        triggerAfterCompletion(defStatus, TransactionSynchronization.STATUS_COMMITTED);
    }
    catch (UnexpectedRollbackException ex) {
        // 7) 예외 시 처리: 애매한 롤백
        triggerAfterCompletion(defStatus, TransactionSynchronization.STATUS_ROLLED_BACK);
        throw ex;
    }
    catch (TransactionException ex) {
        // 8) 커밋 중 오류 발생 시 롤백 & afterCompletion
        doRollbackOnCommitException(defStatus, ex);
        throw ex;
    }
}
```