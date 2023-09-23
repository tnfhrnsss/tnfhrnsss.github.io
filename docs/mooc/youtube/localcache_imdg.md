---
layout: post
title: local cache와 invalidation message propagation 전략을 활용하여 api 성능 튜닝하기 - 강의 정리
date: 2023-06-05 13:10:00
last_modified_at : 2023-06-05 13:10:00
parent: Youtube
grand_parent: Mooc
nav_exclude: true
---

[https://youtu.be/n3fys2E1Lps](https://youtu.be/n3fys2E1Lps)

https://pkgonan.github.io

# 1. Local Cache & Invalidation Message Propagation 전략 도입 배경

- Entity 패턴 분석
    - 우선 자주 호출되는 API 순위를 new relic을 통해 조회해서 어떤 Entity 조회비율이 높은지 보고, 그것들의 전체 트래픽의 얼마나 차지하는지
    - 변경은 자주 되지 않지만 조회가 많이 되는것이면 Cache에 매우 적합한 성격의 Entity라고 판단
- Hibernate 세션 내부에서만 Cache를 공유하는 First Level Cache Hit율 낮고(요청 하나당 유지 되는 개시를 First Level Cache라고 한다.) 여러 세션에서 Cache를 공유하는 Second Level Cache를 적용하면 First Level Cache Miss는 Hit가 가능하게 된다.
    - 그래서 현재 조회 패턴에서 Cache hit율 증가를 위해 Hibernate Second Level Cache를 사용하기로 결정
- Second Level Cache를 쓰면, 같은 엔티티 조회에도 좋다는 것은 알겠는데 그렇다면 Local Cache vs Remote Cache 중 어떤 것이 더 적합한가?
    - 성능 측면 고려
        - 자기 jvm에서 가져오기 때문에 Local Cache가 더 성능이 좋을 수 밖에 없다
    - 일관성 고려
        - Local cache는 일관성을 유지시켜줄 수 없다.
    - 시스템 entity 사용패턴은?
        - 많은 조회가 일어나고 있나?
- 성능과 데이터 일관성 확보를 위해 local cach with imdg를 고려하게 됨
    - imdg?
        - 데이터를 분산 적재 및 처리하고 확장성을 보장하는 솔루션을 말한다.
        - imdg로 설계된 cache 구현체는 entity 상태가 변경되었을 때 변경사항을 다른 인스턴스에게 전파하는 기능을 제공할 수 있다.
        - 그래서 imdg cache 구현체를 local cache로 활용하면 일관성과 성능 모두를 얻을 수 있다.

# 2. Hibernate Cache 동시성 전략과 구현체의 선택

- 어떤 동시성 전략이 적합한가 따져본다.

| Cache Concurrency strategy | 적합성 | 판단근거 |
| --- | --- | --- |
| Read-only | x | 해당 Entity가 수정이 가능한 상황으로 부적합 |
| Nonstrict read/write | o | Entity에 동시 수정이 빈번하지 않으며, 조회가 빈번하여 적합 |
| Read/write | x | Entity수정이 빈번하지 않으며, 스펙상 lock을 제공해야했기 때문에, Lock으로 인한 성능 저하가 발생해서 부적합 |
| Transactional | x | jta를 사용하지 않기 때문에, JTA를 위한 동시성 전략으로 부적합하다고 판단 |
- 따져봤을 때 현상황에서는 Nonstrict read/write 전략이 맞다고 판단하고 그담에 cache구현체를 어떤 것으로 할지 고민합니다.
- 그 결과 invalidation message propagation 기능을 제공하는 imdg 기반 cache 구현체가 필요하다고 판단.
- invalidation message propagation란?

```xml
Cache Invalidation Message Propagation은 캐시의 무효화(invalidation)를 관리하기 위한 전략 중
 하나입니다. 

캐시는 데이터의 불필요한 재검색을 피하기 위해 데이터를 저장하고 재사용하는 기능을 제공합니다.
그러나 데이터가 변경되거나 삭제되었을 때는 캐시된 데이터를 무효화해야 합니다.

Cache Invalidation Message Propagation은 데이터 변경이나 삭제가 발생할 때 캐시에 
무효화 메시지를 전파하는 방법을 의미합니다. 

일반적으로 다음과 같은 절차로 이루어집니다:

- 데이터 변경 또는 삭제가 발생합니다.
- 해당 변경 사항을 알리기 위한 메시지를 생성합니다.
- 메시지를 적절한 메시징 시스템(예: 메시지 큐)에 전송합니다.
- 캐시 서버는 메시지를 수신하고, 메시지에 포함된 데이터에 대한 캐시를 무효화합니다.
- 무효화된 데이터에 대한 다음 요청은 캐시에서 직접적으로 처리되지 않고, 원본 데이터를 다시 가져와 새로운 캐시를 생성합니다.

Cache Invalidation Message Propagation은 분산 시스템에서 캐시를 관리할 때 유용합니다. 
여러 개의 캐시 노드가 존재하고 데이터의 변경이나 삭제가 여러 노드에 영향을 미칠 수 있는 경우, 
메시지 전파를 통해 일관성 있는 캐시 업데이트를 보장할 수 있습니다.

이러한 전략은 캐시의 일관성 유지와 데이터 갱신의 신뢰성을 향상시키는 데 도움이 됩니다. 
그러나 메시지 전파에는 추가적인 시스템 리소스 및 레이턴시가 발생할 수 있으므로, 
효율적인 메시지 전송과 처리 메커니즘을 구현하는 것이 중요합니다.
```

- imdg와의 연관성은?

```xml
IMDG(In-Memory Data Grid)는 데이터를 메모리에 저장하고 처리하는 분산 시스템입니다. 
IMDG는 대량의 데이터를 빠르게 액세스하고 처리하기 위해 여러 개의 노드에 데이터를 분산 저장하며, 
데이터의 일관성과 가용성을 유지합니다.

IMDG는 캐시 기능을 제공하기도 합니다. 
데이터를 메모리에 캐싱하여 빠른 응답 시간과 높은 처리량을 제공하며, 데이터의 재사용성을 높입니다. 

IMDG의 캐시는 캐시된 데이터에 대한 직접적인 액세스를 가능하게 하고, 
데이터의 변경이나 삭제 시 일관성을 유지하기 위해 적절한 캐시 무효화 전략을 사용합니다.

이와 관련하여 IMDG와 Cache Invalidation Message Propagation은 밀접한 관련이 있습니다. 
IMDG는 데이터의 변경이나 삭제가 발생했을 때 캐시의 무효화를 처리하는 기능을 제공합니다. 

변경된 데이터를 IMDG에 업데이트하면 IMDG는 캐시된 데이터를 무효화하고, 
다음 요청 시 새로운 데이터를 캐시에 저장합니다. 

이를 통해 IMDG는 캐시의 일관성을 유지하고 최신 데이터를 제공합니다.

따라서 IMDG는 Cache Invalidation Message Propagation을 지원하여 데이터의 일관성과 무효화를 관리하며, 
빠른 데이터 액세스와 처리를 제공하는데 활용될 수 있습니다.
```

- Local cache & Invalidation message propagation 전략과 cache 구현체 별 적합성 비교
    - 
    
    | 이름 | 구분 | 저장소 | 적합여부 | 판단사유 |
    | --- | --- | --- | --- | --- |
    | Hazelcast | IMDG | In-Memory | o | IMDG로 Invalidation Message Propagation기능 제공 |
    | Infinispan  | IMDG | In-Memory | o | 상동하나 Invalidation Mode에서 Nonstrict_read_write 미지원 |
    | Redis | Non-IMDG | In-Memory | x | Non-Imdg로 invalidation Message Propagation 기능 미제공 |
    | Ehcache | Non-IMDG | In-Memory | x | 상동 |
    | Ehcache & Terracotta | IMDG | In-Memory | x | Terracotta서버 추가 구성으로 인한 부담 |

# 3. Hazelcast 그리고 Second Level Cache

- Hazelcast란
    - imdg이고, 오픈소스이고, 자바로 만들어졌으며, multi thread로 동작한다.
- 배포방식의 선택
    - embedded mode
        - 같은 jvm - gc발생할 수 있음
        - 조회시 network overhead가 없고, serialization overhead도 없다
    - client-server mode
        - 다른 jvm
        - 조회시 network overhead가 발생할 수 있고, serialization overhead도 발생할 수 있음
- hazelcast clustering
    - clustering 방법
        - tcp/ip
        - multicast
        - cloud discovery : plugin을 통해 aws, gcp, azure의 cloud를 호출한다.
- hazelcast 데이터 적재방식
    - 분산 적재
    - 로컬 적재 - 일반적인 local cache와 같으나, update방식만 다르다.
- hazelcast distributed event
    - entity 수정시 클러스터링 된 다른 인스턴스에 전파한다.
    - 전송 방식
        - clustering된 node의 ip를 알고 있다. node를 가져와서 loop돌며 데이터 전송
        - 기본적으로 비동기전송(10만번씩)
    - 실패 처리 방식
        - Local Cache + Invalidation메시지 전파방식은 Retry 정책을 사용
        - 동기 전송 재시도 횟수 50, 비동기 전송 재시도 횟수 5번
- distributed event를 통한 cache 무효화 신뢰성 분석
    - 인스턴스간의 무효화 메시지 도착 시간 차이로 인한 신뢰성
        - 미세한 시간 차이가 존재
        - 시간차로 이슈가 발생하지 않는 성격의 entity를 적재해야한다. ( 즉, strong consistency는 피해야한다)
    - 네트워크 이슈로 인해 cache 무효화 메시지를 받지 못해 생길 수 있는 문제
        - 내부적으로 tcp 전송을 사용하고 있고 패킷 유실등은 프로토콜에서 재처리하고 있기 때문에 여기에 따른 이슈는 없다고 보면 된다.
    - 일시적 500에러로, 로드밸런서에서 인스턴스 제외 후 재 포함된 경우
        - lb통하지 않고, 인스턴스간 직접 통신으로 eviction 정상처리
    - 동기&비동기 이벤트가 최대 재시도 회수 50회, 5회를 초과한 경우
        - 이 상황은 이미 장애상황으로 판단한다.
        - 몇번까지 retry를 할 것인가는 정책으로 한다.

# 4. Cache Hit율 극대화 전략의 적용

- 적용 배경
    - spring jpa에서 2lc는 findById() PK 단건 조회만으로도 HIT가 된다.
    - 그래서 findAllById() PK복수 조회는 기본적으로 cache를 타지 않는다. 그래서 전반적으로 findAllById를 자주 호출할 경우, 캐시를 적용해서 HIT율을 높히는 것이 관건
    - 히트율이 70%이하면 캐시를 안쓰는게 좋다. 한번 read연산을 얼마나 자주하는지 체크하는 것이 좋다. 
    - 히트율 관련해서는 별도의 포스트로 ..

# 5. Local Cache & Invalidation Message Propagation 전략 적용 시점

- result cache가 아닌 entity cache인 경우에만
- 수정보단 entity 조회시
- entity가 1초에 수백 ~수천번 이상 조회될 경우
- entity가 Eventual Consistency에 문제가 없을 경우
- 장단점
    - 장점: 성능에 좋고, 운영측면에서 좋고, 인스턴스 추가에 대한 부담이 없다(embedded일때)
    - 단점: 로컬 캐시라서 확장성이 떨어지고, gc부담이 있다

# 6. 결론

- gc부담 최소화를 위해 적절한 Heap사이즈를 설정하고 Max Cache Entity Size 제한을 통해 Application내의 가용 메모리 유지 가능
- Result Cache 혹은 조회 빈도가 적은 데이터는 Remote Cache를 추천