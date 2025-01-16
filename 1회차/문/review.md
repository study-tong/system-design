### 배운 점 (Soft Skills)

#### 1. 특정 오픈소스 시스템 언급 자제

- 면접에서 Redis나 Kafka와 같은 구체적인 시스템은 **면접관이 먼저 언급**할 때까지 기다리는 것이 좋다.
- 예를 들어, 이번에 구현 용이성을 이유로 Kafka를 사용한다고 언급했는데, "왜 Kafka를 써야 하는가?"라는 질문이 나왔다.
- 특정 기술을 바로 언급하기보다는, 해당 시스템의 **역할과 목적**을 먼저 설명하는 것이 더 좋다.

#### 2. 질문 동기화의 중요성

- 면접관은 웬만하면 지원자의 말을 끊지 않기 때문에, 질문을 잘못 이해한 채로 한참 설명할 가능성이 있다.
- 이를 방지하려면, **면접관과 질문의 정의를 동기화**하면서 문제를 해결해 나가는 것이 중요하다.

---

### 배운 점 (Hard Skills)

#### 1. CDC (Change Data Capture)

- 기존에는 Message Queue(Kafka)를 사용하려면 **Application Layer**에서 이벤트를 직접 발행해야 한다고 생각했다.
- 하지만 **CDC**를 사용하면, 데이터베이스의 로그를 모니터링하여 데이터 변경 사항을 감지하고, 이를 이벤트로 발행할 수 있다.

#### 2. ShedLock

- 여러 대의 노드에서 동작하는 분산 시스템에서, 특정 작업이 동시에 실행되지 않도록 **RDB** 기반으로 Lock을 관리할 수 있다.
- **ShedLock**은 이를 라이브러리로 구현한 것으로, 주로 Spring 환경에서 사용된다.

##### ShedLock에 필요한 필드

- **name**: Lock의 식별자.
- **lock_until**: Lock이 유지되는 시간.
- **lock_at**: Lock이 획득된 시간.
- **lock_by**: Lock을 소유한 노드 또는 인스턴스.

### Reference 정리(by 진호님)

---

#### **1. CDC + Kafka**

- **출처**: [CDC 너두 할 수 있어(feat. B2B 알림 서비스에 Kafka CDC 적용하기) | 우아한형제들 기술블로그](https://techblog.woowahan.com/10000/)
- **CDC의 데이터 변경 감지 방식**
    1. **Pull**: 폴링 방식으로 주기적으로 데이터를 요청.
    2. **Push**: 데이터 변경 시 타깃 시스템에 즉시 알림.
        - **장점**: 실시간성이 뛰어남.
        - **단점**: 소스 시스템 작업량 증가, 타깃 시스템 장애 시 이벤트 누락 가능성.
- **Kafka 활용**:
    - 이벤트 누락 문제를 완화.
    - **Debezium MySQL Connector**를 이용해 MySQL의 binlog를 읽고 변경 이벤트를 Kafka 토픽으로 전송.

---

#### **2. 알림은 NoSQL이 적합한 이유**

- **출처**: [LINE 알림 센터의 메인 스토리지를 Redis에서 MongoDB로 전환하기](https://engineering.linecorp.com/ko/blog/LINE-integrated-notification-center-from-redis-to-mongodb)

1. **데이터 구조**:
    - 알림 데이터는 하나의 객체로 처리되며, 부분적으로 사용되지 않음.
    - 데이터 간 독립성이 높아 정규화가 불필요.
2. **스키마 변화 가능성**:
    - 알림 스펙이 자주 변경되고 기능 추가 가능성이 큼.
    - **스키마리스(schemaless)** 구조가 유리.
3. **NoSQL의 장점**:
    - 데이터 추가 및 변경이 자유로움.
    - 변화하는 스펙에 대한 대응이 용이.

---

#### **3. RDB vs NoSQL (알림 도메인 관점)**

- **출처**: [Java 기반의 알림 서비스로 MSA 전환기 - Remember & Company](https://blog.dramancompany.com/2022/01/java-%ea%b8%b0%eb%b0%98%ec%9d%98-%ec%95%8c%eb%a6%bc-%ec%84%9c%eb%b9%84%ec%8a%a4%eb%a1%9c-msa-%ec%a0%84%ed%99%98%ea%b8%b0/)

1. **TTL(Time to Live)**:
    - 알림은 일정 기간(예: 한 달) 이후 삭제되므로 영구 저장이 불필요.
    - **TTL 기능**을 기본 지원하는 NoSQL이 적합.
2. **유연한 스키마 대응**:
    - 읽기/쓰기 빈도가 높은 알림 도메인에서는 **schema-less** 구조가 유리.
    - 알림 행위를 정의하는 구조는 자주 변경되므로 유연성이 필요.
3. **성능**:
    - 알림 도메인에서는 트랜잭션 관리가 필요하지 않음.
    - **NoSQL은 트랜잭션 관리 없이도 높은 성능** 제공.

---

#### **4. FCM 튜닝 및 발행 보장**

- **출처**:
    - [FCM과 함께 춤을(튜닝)](https://kwj1270.tistory.com/entry/Spring-FCM-%ED%86%BA%EC%95%84%EB%B3%B4%EA%B8%B0-2-%ED%8A%9C%EB%8B%9D)
    - [FCM과 함께 춤을(발행 보장)](https://kwj1270.tistory.com/entry/FCM%EA%B3%BC-%ED%95%A8%EA%BB%98-%EC%B6%A4%EC%9D%84%EB%B0%9C%ED%96%89-%EB%B3%B4%EC%9E%A5)
1. **튜닝**:
    - FCM 서버의 성능을 최적화하기 위해 메시지 크기, 요청 간격, 재시도 횟수 등 세부 설정 필요.
2. **발행 보장**:
    - 장애 발생 시에도 안정적으로 메시지를 발송하기 위해 재시도 정책, Dead Letter Queue(DLQ) 설정 필수.