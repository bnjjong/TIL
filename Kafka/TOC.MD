## 📖 1주차: 기초 이해 및 환경 구축 (1–5회차)

1. **스터디 개요 및 목표 설정**

   * 학습 목표, 운영 환경(AWS MSK), 도메인(광고 통계) 정리
   * 전체 로드맵과 평가 지표 수립

2. **Kafka 기본 개념**

   * 메시지 큐 vs. 스트리밍 플랫폼
   * 토픽(Topic), 파티션(Partition), 오프셋(Offset)의 의미

3. **Kafka 아키텍처 심화**

   * 브로커(Broker), 리더/팔로워 복제(Replication)
   * 프로듀서(Producer)와 컨슈머(Consumer) 동작 원리

4. **AWS MSK 개요 및 설정**

   * MSK 클러스터 생성, VPC/서브넷 구성
   * MSK 보안 그룹, IAM 권한, TLS 설정

5. **로컬 개발 환경 구축**

   * Docker Compose로 Kafka 단일/클러스터 모드 실행
   * Schema Registry, Zookeeper(or KRaft) 간단 데모

---

## 🔌 2주차: Spring Boot 연동 및 메시징 실습 (6–10회차)

6. **Spring Kafka 기초**

   * `spring-kafka` 의존성, 주요 프로퍼티
   * `KafkaTemplate` 과 `@KafkaListener` 개념

7. **Producer 실습**

   * 간단한 광고 이벤트 발행 예제
   * 콜백, 에러 핸들링, 동기 vs 비동기 전송

8. **Consumer 실습**

   * Consumer Group 설정과 파티션 할당 전략
   * 메시지 수신 후 통계 집계 로직 구현

9. **토픽 및 파티셔닝 전략**

   * 파티션 수 결정 기준(병렬 처리량 vs. 오더링)
   * 키 기반 파티셔닝 vs Round-Robin

10. **직렬화/역직렬화**

    * JSON vs Avro vs Protobuf 비교
    * Confluent Schema Registry 연동

---

## ⚙️ 3주차: 운영·성능 튜닝 (11–15회차)

11. **프로듀서 성능 튜닝**

    * 배치 크기(`linger.ms`, `batch.size`)
    * 압축(`compression.type`)과 직렬화 비용

12. **컨슈머 성능 튜닝**

    * 폴링 간격(`max.poll.interval.ms`, `fetch.min.bytes`)
    * 오프셋 커밋 전략(자동 vs 수동)

13. **클러스터 레벨 튜닝**

    * 브로커 설정(`num.io.threads`, `log.segment.bytes`)
    * 디스크와 네트워크 튜닝 포인트

14. **모니터링 및 로깅**

    * JMX → Micrometer → Prometheus → Grafana 연동
    * 주요 메트릭: 처리량, 대기 시간, ISR 상태

15. **운영 자동화**

    * Terraform/CloudFormation으로 MSK 프로비저닝
    * 롤링 재시작, 스케일 아웃/인 전략

---

## 🛡️ 4주차: 실패 복구 및 고급 기능 (16–20회차)

16. **장애 처리 전략**

    * Dead-Letter Queue 설계
    * 재시도(Backoff)와 지연 큐 패턴

17. **데이터 정확성 보장**

    * Idempotent Producer
    * 멀티 파티션 트랜잭션(Transaction API)

18. **보안 설정**

    * TLS 암호화, SASL 인증
    * ACL(Role-based Access Control)

19. **Kafka Streams 기초**

    * 토폴로지(Topology) 설계
    * 스트림 집계 및 윈도잉(Windowing)

20. **실전 광고 통계 처리 예제 & 회고**

    * 광고 클릭/노출 로그 파이프라인 설계
    * 병목 분석, 추가 학습 로드맵 수립

---

> 💡 **Tip**:
>
> * 매 세션마다 간단한 미니 실습을 포함하면 이해가 더 빠릅니다.
> * 주요 개념 학습 후 실제 AWS MSK 환경에서 테스트해 보세요.
> * 모니터링 대시보드를 미리 구성해 두면 성능 개선 효과가 눈에 보입니다.

이 목차를 바탕으로 매일 1시간씩 꾸준히 진행하시면, 광고 통계 처리에 최적화된 Kafka 운영 및 개발 역량을 갖추실 수 있을 거예요. 화이팅입니다! 😊
