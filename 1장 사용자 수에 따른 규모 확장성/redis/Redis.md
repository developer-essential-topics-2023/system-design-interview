# Redis

## 캐시

나중의 요청에 대한 결과를 미리 저장했다가, 빠르게 사용하는 것

- 일반적으로 RAM(Random Access Memory)과 같이 빠르게 액세스할 수 있는 하드웨어에 저장된다.
- 캐시로 액세스하므로, 더 느린 기본 스토리지 계층에 액세스해야 하는 필요를 줄임으로써 데이터 검색 성능을 높인다.
- TTL(Time To Live)과 같은 제어 항목을 적용하여 이에 따라 데이터를 만료되도록 할 수 있다.

### 데이터 베이스보다 응답 시간이 빠른 이유

⇒ 인 메모리 스토어 이므로

기존 데이터베이스와 달리 인 메모리 데이터 스토어에는 디스크로의 이동이 필요하지 않기 때문에 엔진 대기 시간이 마이크로초 단위까지 줄어든다.

따라서, 인 메모리 데이터 스토어는 더 많은 작업을 처리하고 더 빠른 응답 시간을 지원할 수 있다.

**메모리 계층 구조**

![Untitled](Redis%2093fd178e387d4affb958a3f5b466dd88/Untitled.png)

## 캐시 : Memcached와 Redis

캐시로는 Memcahed와 Redis를 많이 사용한다.

### **공통점**

- in-memory cache이다.
- key-value 저장소이다.
- ~~데이터관리를 위해 NoSQL을 사용한다~~
- RAM에 데이터를 보관한다

### **Memcached의 특징**

- 직렬화된 형태로 저장하게 제한되어 있어, 정적 데이터 캐싱에 효과적이다.
    - HTML같은 작은, 정적 데이터를 캐싱할 때 효율적이다.
    - 직렬화 오버헤드를 줄일 수 있다.
- 멀티 스레드를 지원한다.
    - 수직적 확장에 유리하다.
    - 하지만 캐시된 데이터를 유실 할 확률도 높아진다.
    - 반면, Redis는 싱글스레드 이므로, 데이터 손실없이 수평으로 스케일링할 수 있다.

### 그런데 왜 Redis를 많이 선택할까?

- 다양한 자료구조, 용량
    - 특히, Collection을 제공한다.
- 다양한 삭제 정책
- 디스크 영구 저장이 가능하다 : 디스크 영속화 지원
- Replication 지원 : 하나의 인스턴스로부터 또다른 레플리카 인스턴스를 복사할 수 있다.
    - Redis는 하나의 이상의 레플리카를 가질 수 있다.
    - Memcached는 서드 파티를 이용해야한다.
- 트랜잭션을 지원한다.
- 더 나은 read 시간 및 메모리 사용 효율성을 가진다.

## Collection

### 장점

1. **개발 편의성이 높아진다.**
    
    랭킹서버를 구현한다고 가정해보자.
    
    일반적인 DB라면 order by를 통하여 정렬시킨 값을 가져와야한다. 하지만 결국 개수가 많아지면 디스크를 사용하기 때문에 속도 문제가 발생하게 된다.
    
    여기서 Redis의 Sorted Set을 통하여 쉽게 구현 할 수 있다.
    
    ![스크린샷 2023-05-11 오후 7.47.58.png](Redis%2093fd178e387d4affb958a3f5b466dd88/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-05-11_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_7.47.58.png)
    
2. **개발의 난이도가 낮아진다.**
    
    DB에서 트랜잭션이 동시에 수행되어, Race Condition이 일어나 잘못된 데이터가 덮어씌워져 저장될 수 있다.
    
    하지만 **Redis는 싱글 스레드이기 때문에 자료구조가 Atomic 하므로**, Race Condition을 피할 수 있다. 물론, 무조건 일어나지 않는 것은 아니다.
    
    따라서 Collection을 이용해 비즈니스 로직에 집중하면되므로, 개발의 난이도가 낮아진다.
    

### 주의 사항

- 하나의 컬렉션에 너무 많은 아이템을 담으면 비효율적이다.
    - 10,000개 이하 몇천개 수준으로 유지하라.
- Expire는 Collection의 개별 item에 걸리지 않고 전체 Collection에만 걸려서 모두 삭제될 위험이 있다.
    - 즉, 해당 10,000개의 아이템을 가진 Collection에 expire가 걸려있다면, 그 시간 후에 10,000개의 아이템이 모두 삭제되는 것이다.

## Redis

Remote dictionary server

빠른, 오픈 소스, 인 메모리, 키 값 데이터 스토어

### 특징

- Database
- 캐시
- Message broker
- 메모리상에 데이터를 저장한다
- 다양한 자료 구조 제공
    - 문자열 – 최대 512MB 크기의 텍스트 또는 바이너리 데이터
    - List – 추가된 순서가 유지되는 문자열 모음
    - Sets – 순서가 유지되지 않는 문자열 모음으로 다른 세트 유형과 교차, 통합 및 비교 가능
    - Sorted Sets – 값을 기준으로 순서가 지정된 세트
    - Hashes – 필드 및 값의 목록을 저장하는 데이터 구조
    - Bitmaps – 비트 수준 작업을 제공하는 데이터 유형
    - HyperLogLogs – 데이터 집합 내 고유 항목을 추정하기 위한 확률적 데이터 구조
    - Streams - 로그 데이터 구조 메시지 대기열
    - Geospatial - 경도/위도 기반 항목 맵, ‘인근’
    - JSON - 숫자, 문자열, 부울, 배열 및 기타 개체를 지원하는 명명된 값의 중첩된 반정형 객체

### 사용 사례

- 인증 토큰 저장 : String / hash / …
- 랭킹 서버 사용 : Sorted Set / …
- 유저 API Limit
- Job Queue : List / …

## Redis는 싱글 스레드

### 제약

긴 작업을 처리하는 동안, 다른 요청을 처리하지 못한다. 

- 긴 작업 : O(N)의 명령 수행 대상이 아주 많은 것

### 장점

- 멀티 스레드를 사용하면서 발생하는 context-switching 비용을 줄일 수 있다.
- 스레드간 자원 공유 제약이 없다.

### 왜 싱글 스레드인데, 성능이 좋을까?

- cpu-intensive 하지 않아 (cpu 중심적이지 않아), 싱글스레드 만으로 초당 10만건 이상의 데이터 요청을 처리할 수 있다.
    - cpu 위주가 아닌, 메모리 I/O 작업이 대부분 이므로, CPU 최적화는 것은 효율이 떨어진다.
    - 다중 프로세스 전략 즉 클러스터 전략을 통해 성능을 개선할 수 있다.
- 이벤트 루프는 단일 스레드이지만, 내부적으로 이벤트를 처리하는 IO는 I/O Multiplexing을 통해 멀티스레드로 동작한다.
    
    ![Untitled](Redis%2093fd178e387d4affb958a3f5b466dd88/Untitled%201.png)
    
    - 마치 node.js 처럼 동작하는 것이다.

### 참고 : Node.js는 싱글스레드이면서도 아니다 - 싱글스레드에서 비동기 작업 처리

자바스크립트는 싱글 스레드 언어인데, 어디서 비동기 작업들을 처리하고 있는걸까? 바로 이벤트 루프이다. 

![Untitled](Redis%2093fd178e387d4affb958a3f5b466dd88/Untitled%202.png)

![Untitled](Redis%2093fd178e387d4affb958a3f5b466dd88/Untitled%203.png)

Node.js는 자바스크립트 기반의, Event Loop의 싱글 스레드를 가진 언어이나, 멀티 스레딩을 지원하는 C언어 라이브러리 libuv 를 사용한다. Node.js의 libuv는 기본적으로 4개의 스레드를 가진 스레드풀을 가지고 있다. 

따라서, 동기적인 작업은 이벤트 루프에서 작업하며, 비동기적인 작업은 스레드 풀에서 작업한다.

**결론적으로, Node.js는 싱글스레드의 자바스크립트 기반 언어이므로, 기본적으로는 싱글스레드 언어이나, 비동기 작업을 수행할 때에는 멀티 스레드 프로세스이다.**

## 메모리 관리

### 메모리 파편화

메모리를 할당받고 해제하는 과정에서, 비어있는 공간이 생긴다. 

이러한 메모리 파편화 때문에 Max Memory 설정을 하여도 그 메모리 보다 더 많은 공간을 사용하게 된다.

따라서 다양한 사이즈를 가지는 데이터 보다는 유사한 크기의 데이터를 가지는 경우가 좋고, 큰 메모리 인스턴스 하나보다는 적은 메모리 인스턴스 여러개가 안전하다.

![스크린샷 2023-05-11 오후 9.24.52.png](Redis%2093fd178e387d4affb958a3f5b466dd88/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-05-11_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_9.24.52.png)

### 가상 메모리 Swap

Swap이 있다면, 해당 메모리 Page를 접근할 때마다 늦어지는 문제가 있음

따라서 Physical memory를 얼마나 쓰고 있는지의 지표인 RSS 값을 모니터링해야함

### O(N) 관련 명령어

Redis는 싱글 스레드이므로 Buffer에 쌓인 요청을 하나씩 빠르게 처리해야 한다. 따라서 처리 하는데 시간이 오래걸리는 요청이 온다면 뒤에 쌓인 요청들은 모두 대기해야 하므로 O(N) 관련 명령어는 피해야 한다.

![Untitled](Redis%2093fd178e387d4affb958a3f5b466dd88/Untitled%204.png)

- KEYS
- FLUSHALL
- FLUSDB
- Delete Collections
- Get All Collections

등이 있다.

## 레퍼런스

[[우아한테크세미나] 191121 우아한레디스 by 강대명님](https://www.youtube.com/watch?v=mPB2CZiAkKM)

[Redis](https://www.slideshare.net/charsyam2/redis-196314086)

[Redis: 인 메모리 데이터 스토어 사용 방법 및 필요성](https://aws.amazon.com/ko/redis/)