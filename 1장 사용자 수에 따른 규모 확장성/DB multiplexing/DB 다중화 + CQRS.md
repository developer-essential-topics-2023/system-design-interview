# DB 다중화 + CQRS

DB 서버가 **데이터를 보존**하는 **영속(persistence)** 계층이기 때문에 웹 서버나 애플리케이션 서버와 비교했을 때 다중화에 대해 고민해야할 부분이 많다.

![Untitled](../DB%20multiplexing/DB%20%E1%84%83%E1%85%A1%E1%84%8C%E1%85%AE%E1%86%BC%E1%84%92%E1%85%AA%20+%20CQRS%20977efa00dcc649eda66bacff3fe037c1/Untitled.png)

위의 사진처럼 DB 서버와 저장소가 각각 한 대로 구성되어 있다고 해보자. 만약 DB 서버가 다운되면 관련 서비스 전체가 중단되어버릴 것이다. 

서비스 중단을 막기 위한 간단한 방법은 DB 서버를 다중화하는 것이다. 이 경우 저장소는 1개이므로 데이터 정합성에 대해서는 신경 쓸 필요 없다.

# 클러스터링 ****Clustering****

![Untitled](../DB%20multiplexing/DB%20%E1%84%83%E1%85%A1%E1%84%8C%E1%85%AE%E1%86%BC%E1%84%92%E1%85%AA%20+%20CQRS%20977efa00dcc649eda66bacff3fe037c1/Untitled%201.png)

**DB 서버의 다중화** 클러스터링은 2대의 DB 서버가 동시에 가동되는지에 따라 2가지로 분류한다.

1)  Active-Active

2) Active-Standby

## **Active-Active**

클러스터를 구성하는 컴포넌트를 동시에 가동

- 장점
    - 복수의 DB 서버가 동시에 동작하고 있어 1대가 다운되어도 남은 서버가 계속 처리 가능하다.
    - DB 서버 대수가 증가하면서 동시에 가동되는 CPU나 메모리가 증가하므로 처리 성능이 향상된다.
- 단점
    - 저장소가 병목 지점이 되어 성능 문제를 일으킬 수 있다.

## **Active-Standby**

클러스터를 구성하는 컴포넌트 중 실제로 가동하는 것은 Active이고, 남은 하나는 대기(Standby)하고 있다가 장애 발생 시 가동

- 종류
    - **Cold-Standby:** 평소에는 Standby DB가 작동하지 않다가 Active DB가 다운된 시점에 작동
    - **Hot-Standby**: 평소에도 Standby DB가 작동
- 장점
    - Active-Active의 저장소 병목 현상이 해결된다.
- 단점
    - Failover(예비 시스템으로서의 자동 전환)가 이뤄지는 전환 시간 동안에는 서비스가 불가능해진다.
    - Hot-Standby은 전환 시간이 짧지만, 그만큼 라이센스료가 높다. 실제로 동작하는 것은 Active DB 1대이지만 전환 시간을 줄이기 위해 Hot-Standby의 높은 라이센스료를 지급한다는 것은 사치스러운 구성이다.

### ✅ **Standby DB 서버는 어떻게 Active DB 서버에서 장애가 일어난 것을 알아챌까?**

Standby DB 서버는 일정 간격으로 Active DB에 이상이 없는지 조사하기 위한 통신을 하는데 이 통신을 ‘Heartbeat’라 한다. Active DB에서 장애가 발생하면 이 신호가 끊기기 때문에 ‘죽었다’라고 인지하게 된다.

### ✅ **Failover**

실패(Fail)를 끝내는(over) 것으로 **장애 대비 기능**을 의미한다. 시스템에 장애가 오면 미리 준비했던 다른 시스템으로 대체해서 운영하는 것이다.

운영중인 서버(Active)에 문제가 생기면 대기 상태에 있던 Standby 서버가 Active 상태로 변경되면서, 서비스를 이어 운영한다. 즉, Failover 기능을 통해 두 서버의 상태를 상호 전환함으로써 장애를 대응하는 방법이다.

### **라이센스에 따라 가용성과 성능이 뛰어난 순서**

1. Active-Active
2. Active-Standby(Hot-Standby)
3. Active-Standby(Cold-Standby)

### 정리

Active-Active 구조는 DB 서버의 부하를 분산시키지만, 저장소가 병목 지점이 된다는 단점이 있다. Active-Standby 구조로 병목 현상을 해소 시킬 수 있지만, failover가 이뤄지는 동안에는 서비스가 불가능하다는 단점이 있다. 

Active-Active와 Active-Standby 클러스터 구성에서는 DB 서버 부분은 다중화하지만 저장소 부분은 다중화하지 않는다는 공통적인 단점이 있었다. 이는 저장소에 문제가 생기면 데이터를 잃게 될 것을 의미한다. 이런 상황에 대응할 수 있는 클러스터 구성이 리플리케이션(Replication)이다.

# 리플리케이션 Replication

리플리케이션은 **DB 서버 뿐만 아니라 데이터도 다중화**하는 것으로, DB 서버와 저장소를 세트로 하여 복수로 준비하는 것이다.

Master DB 와 Slave DB를 복제하여 데이터를 분산시키는 방법으로, Master DB 는 데이터를 저장하고, Slave DB는 Master DB의 데이터를 복제하여 항상 최신 데이터를 유지한다.

### Master-Slave 간의 동기화 방법

해당 구조에서 주의할 점은 Standby(Slave) 측 데이터의 최신화를 유지해야한다는 점이다. 즉, Master DB와 Slave DB 간의 동기화를 일정 주기로 수행해 나가야 하는데, Slave 측 갱신 주기를 얼마로 할 것인가와 성능 사이에 Trade-off 관계가 생긴다.

# Shared Nothing

복수의 서버가 한 개의 저장소를 공유하는 것을 Shared Disk, 네트워크 이외의 자원을 아무것도 공유하지 않는 방식을 Shared Nothing 구조라 한다.

![Untitled](../DB%20multiplexing/DB%20%E1%84%83%E1%85%A1%E1%84%8C%E1%85%AE%E1%86%BC%E1%84%92%E1%85%AA%20+%20CQRS%20977efa00dcc649eda66bacff3fe037c1/Untitled%202.png)

앞선 Active-Active 클러스터 구성에서 저장소 부분이 병목되는 경우가 있는데, 이는 복수의 서버가 1대의 저장소를 공유(Shared Disk)하기 때문이다. 이를 해결하기 위해 고안된 것이 Shared Nothing이다.

Shared Nothing에서는 DB 서버와 저장소로 구성된 세트를 늘려서 저장소가 병목 지점이 되는 것을 방지한다. 또한 병렬처리로 인해 선형적으로 성능이 향상된다. 구글에서 개발한 Shared Nothing 구조로 ✨**샤딩(Sharding)**✨이 있다.

하지만, 저장소를 공유하지 않는 것은 결국 **각각의 DB 서버가 동일한 1개의 데이터에 접근할 수 없다**는 것을 의미한다. 이런 문제에 대처하기 위해 DB 서버 하나가 다운되었을 때 다른 DB 서버가 이를 이어받아 계속 처리할 수 있게 하는 커버링(Covering) 구성 등을 고려해야 한다.

# DB 아키텍처 패턴 정리

![출처 - [https://hololo-kumo.tistory.com/220](https://hololo-kumo.tistory.com/220)](../DB%20multiplexing/DB%20%E1%84%83%E1%85%A1%E1%84%8C%E1%85%AE%E1%86%BC%E1%84%92%E1%85%AA%20+%20CQRS%20977efa00dcc649eda66bacff3fe037c1/Untitled%203.png)

출처 - [https://hololo-kumo.tistory.com/220](https://hololo-kumo.tistory.com/220)

## Refernce

- [https://hololo-kumo.tistory.com/220](https://hololo-kumo.tistory.com/220)
- [https://yjna2316.github.io/database/2020/12/12/DB-archi-다중화(클러스터링)](https://yjna2316.github.io/database/2020/12/12/DB-archi-%EB%8B%A4%EC%A4%91%ED%99%94(%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0%EB%A7%81)/)
- [https://yjna2316.github.io/database/2020/12/12/DB-archi-다중화(Replication과-Shared-Nothing)](https://yjna2316.github.io/database/2020/12/12/DB-archi-%EB%8B%A4%EC%A4%91%ED%99%94(Replication%EA%B3%BC-Shared-Nothing)/)
- [https://yjna2316.github.io/database/2020/12/13/Sharding](https://yjna2316.github.io/database/2020/12/13/Sharding/)
- [https://kgh940525.tistory.com/entry/Database-DB-서버의-다중화Multiplexing-클러스터링-리플리케이션Replication](https://kgh940525.tistory.com/entry/Database-DB-%EC%84%9C%EB%B2%84%EC%9D%98-%EB%8B%A4%EC%A4%91%ED%99%94Multiplexing-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0%EB%A7%81-%EB%A6%AC%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98Replication)

---

# CQRS 패턴

> Command and Query Responsibility Segregation의 약자로, Command(명령- insert, update, delete)와 Query(조회 - select)를 분리하는 아키텍처 패턴
> 
- 장점
    - 도메인 로직에만 집중할 수 있게 된다
    - 데이터소스의 독립적인 크기 조정이 가능하다
    - 읽기와 쓰기를 분리함으로써 보안 관리가 용이
- 단점
    - 시스템의 복잡성이 올라간다
    - 즉시적인 일관성이 보장되지 않는다
        - 쓰기 모델과 읽기 모델의 100% 동기화를 위해서는 추가적인 모델을 사용하거나 위에서 언급했던 별도의 동기화 전략이 필요

내용 추가할 예정