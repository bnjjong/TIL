|  일차 | 주제                       | 세부 내용                                                                    |
| --: | ------------------------ | ------------------------------------------------------------------------ |
|  D1 | 트랜잭션 기초 및 ACID 복습        | - 트랜잭션 정의, ACID 특성<br>- DB 트랜잭션 vs 애플리케이션 트랜잭션                           |
|  D2 | Spring 트랜잭션 아키텍처         | - `PlatformTransactionManager` 역할<br>- `TransactionInterceptor`와 AOP 프록시 |
|  D3 | `@Transactional` 기본 사용법  | - 어노테이션 위치(클래스/메서드)<br>- 읽기전용(readOnly), 타임아웃 설정                         |
|  D4 | 전파(Propagation) 속성 완전 정복 | - `REQUIRED`, `REQUIRES_NEW`, `NESTED` 등 7가지 전파 모드 비교<br>- 예제 실습         |
|  D5 | 격리(Isolation) 레벨 심화      | - `READ_UNCOMMITTED`\~`SERIALIZABLE` 이해<br>- 데모 DB에서 교착/팬텀 테스트           |
|  D6 | 롤백(Rollback) 규칙과 예외 처리   | - 기본 롤백(On RuntimeException)<br>- `rollbackFor`·`noRollbackFor` 활용       |
|  D7 | 프로그래밍적 트랜잭션 관리           | - `TransactionTemplate` 사용법<br>- `TransactionCallback` 예제                |
|  D8 | 트랜잭션 동기화와 리스너            | - `TransactionSynchronization` 인터페이스<br>- `@TransactionalEventListener`  |
|  D9 | 중첩(Nested) 트랜잭션과 저장점     | - Savepoint 생성/롤백 흐름<br>- JDBC vs JTA 차이                                 |
| D10 | 다중 데이터소스·분산 트랜잭션         | - `ChainedTransactionManager` 개념<br>- XA 트랜잭션 기초                         |
| D11 | 트랜잭션 테스트 전략              | - `@Transactional` 테스트<br>- `TestTransaction` API 활용                     |
| D12 | 성능·모니터링·장애 대응            | - 슬로우 쿼리 모니터링<br>- 트랜잭션 로그·AOP 포인트컷 분석                                   |
| D13 | Reactive·리액티브 트랜잭션       | - `TransactionalOperator`<br>- R2DBC 연동 실습                               |
| D14 | 종합 실습 & 코드 리뷰            | - 간단한 미니 프로젝트에 트랜잭션 적용<br>- 스터디 회고 및 추가 학습 로드맵                           |
