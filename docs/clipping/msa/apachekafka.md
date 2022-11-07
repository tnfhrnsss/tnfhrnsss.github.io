---
layout: post
title: 실전 아파치 카프카 정리
date: 2022-05-21 18:51:00
last_modified_at : 2022-11-07 20:46:00
has_children: false
parent: Clipping
categories: kafka
tags: ['msa', 'kafka']
nav_exclude: true
---

## 예제 소스

http://www.hanbit.co.kr/src/10280

## 소개

카프카를 처음 접하는 경우 읽으면 개략적으로 용도 및 활용 범위 등을 알 수 있는 입문서입니다.

## summary

- 카프타 전달 보증 수준
    - At Most Once : 메시지는 중복되지 않지만 상실될 수도 있다. 1회는 전달을 시도해본다. 재전송x
    - At Least Once : 메시지가 중복될 가능성은 있지만, 상실되지는 않는다. 적어도 1회는 전달한다. 재전송o
    - Exactly Once : 중복되거나 상실되지도 않고, 확실하게 메시지가 도달하지만, 성능이 나오기 힘들다. 1회만 전달한다. 재전송o
        - 프로듀서와 브로커의 상호 교환 사이를 살펴보면 양쪽 모두에게 시퀀스 번호를 관리해 중복되는 실행을 제거하는 방법을 사용한다 = 멱등성을 유지

- 스키마 호환성
    - schema registry는 대수를 늘려도 확장 구성이 되지 않는다 2개 이상이면 충분
    - schema registry도 restapi 가 있다.
    - 후방(하위)호환성  - backward compatibility
        - 예전 스키마의 데이터를 새로운 스키마를 사용해서도 로딩할 수 있다는 성질
    - 전방(상위)호환성 - forward compatibility
        - 새로운 스키마의 데이터를 예전 스키마를 사용해서도 로드할 수 있는 성질
    - 완전 호환성 - full compatibility
        - 후방 호환성과 전방 호환성을 모두 갖춘 경우

- 스트림의 처리 기본
    - 간헐적으로 유입되는 데이터를 수시로 처리하는 처리 모델이다
        
        ```java
        while(running) {
        	ConsumerRecords<String, String> records = consumer.poll(1000);
        	for (ConsumerRecord<String, String> record : records) {
        		System.out.println(record.value());
        	}
        }
        ```
        
    - 스트림 처리 오류 다루기
        - 동일한 패턴에서의 예외처리가 여러 곳에 있는 경우, 그것을 대응하는 람다식 함수를 래핑해, 예외가 발생했을 경우 빈 목록을 반환하는 함수로 변환하는 메서드를 정의해서 사용한다.
            - stream에서 wrap메서드 호출을 통해 공통 처리
                
                ```java
                // 호출
                stream.flatMapValues(wrap(v -> v.substring(3)))
                
                // interface
                @FunctionalInterface
                private interface FunctionWithException<T, R, E extends Exception> {
                	R apply(T t) throws E;
                }
                
                private static <V, VR, E extends Exception> ValueMapper<V, Iterable<VR>> wrap(FunctionWithException<V, VR, E> f) {
                	return new ValueMepper<V, Iterable<VR>>() {
                		public Iterable<VR> apply(V v) {
                			try {
                				return Arrays.asList(f.apply(v));
                			} catch(Exception e) {
                				e.printStackTrace();
                				return Arrays.asList();
                			}
                		}
                	}
                }
                ```
                
- 컨슈머에서 파티션 할당
    - 컨슈머와 파티션의 매핑은 각 파티션에 하나의 컨슈머가 매핑된다. 반대로 파티션 수에 따라 하나의 컨슈머에 여러 파티션이 할당되는 경우가 있다.
    - 어사이너 assignor
        - 각 파티션을 컨슈머 그룹 내에 있는 특정 컨슈머에 맵핑할 것인가에 대해서 결정하는 로직
        - 종류
            - roundrobin : 매핑할 파티션을 컨슈머에 하나씩 차례대로 매핑한다.
            - range : 매핑할 파티션을 나열하고 컨슈머 수로 영역을 분할하여 할당한다.
            - sticky : 최대한 균형있게 할당하고 재할당 시에는 원래의 매핑에서 변경되지 않도록 할당한다.
        - KafkaConsumer를 생성할 때 옵션에서 설정할 수 있다
            
            ```kotlin
            partition.assignment.strategy
            ```
            
- KSQL
    - 스트림과 테이블은 KSQL에서 다루는 구조화 데이터를 위한 것으로 스트림이 연속적으로 발생하는 구조화 데이터를 취급하는 반면, 테이블은 스트림이나 다른 테이블에 대해 집계한 데이터를 취급한다.
- 파티션 수 설정시 고려사항
    - 카프카 클러스터의 메시지 송수신 측면
        - 브로커 수에 비해 파티션 수가 적으면 특정 브로커에만 리더 복제본이 존재하게 되어 부하가 몰리게 된다.(리더 복제본은 팔로워 복제본에 비해 부하가 올라가기 쉽다)
        - 브로커를 늘릴 때는 새롭게 증가하는 브로커에도 리더 복제본을 배치해서 각 브로커를 균등하게 처리하기 위해 더 많은 파티션이 필요하다.
    - 컨슈머 그룹의 할당
        - 파티션 수가 적어도 각 컨슈머 그룹에 속하는 컨슈머보다 많아야 메시지를 분산해서 수신할 수 있다.
    - 브로커가 이용하는 디스크
        - 카프카는 데이터를 되도록이면 순차적으로 기재해야 하기 때문에 되도록이면 브로커가 사용하는 디스크는 RAID, JBOD 등의 메커니즘을 통하지 않고 직접 사용하는 것이 바람직하다.
        - 전체 브로커에서 모든 디스크를 효율적으로 사용하기 위해서는 디스크 수 이상의 파티션 수가 카프카 클러스터 전체에서 필요하다
- 복제본 수(replication-factor) 결정 시 고려사항
    - 복제본은 내장애성을 위한 구조로 되어있다. 그래서 replication-factor는 카프카 클러스터가 어느 정도의 장애에 견딜 수 있는가를 결정하는 중요한 설정이다.
    - 브로커의 장애 허용 대수와의 관계를 고려한다
        - min.insync.replicas의 설정
            - 프로듀서가 메시지를 보낼 때 송신처 파티션의 복제본 중 isr에 속하는 복제본이 최소 몇 개나 필요한지를 설정하는 브로커와 토픽의 구성이다.
            - (재배치 수) ≥ (min.insync.replicas) + (브로커의 장애 허용 대수)
                - 장애 허용 대수만큼의 브로커에 장애가 발생했을 때 서비스를 계속하기 위해서는 모든 파티션에서 ISR에 속하는 복제본의 수가 이 min.insync.replicas수 이상이어야 한다.
    - 토픽을 작성할 때 live broker 대수
        - (재배치 수) ≤ (브로커 총수) - (브로커의 장애 허용 대수)
        - 제대로 동작하고 있는 브로커(live broker)의 수가 재배치 수 이상 존재하지 않으면 토픽 작성에 실패한다.
- 정리
    - kafka connect
        - 외부데이터를 카프카로 입력하고 카프카에서 외부로 데이터 출력 기능을 제공한다.
        - 카프카 커뮤니티에서 개발한 도구로 카프카 패키지에 포함되어있다.
        - confluent가 제시하는 디자인 패턴
            - kafka connect로 외부에서 데이터를 송신하고 → kafka streams를 이용해서 필요한 데이터를 처리하고 처리결과를 카프카에 송신하면 → kafka connect를 이용해서 연계 시스템에 데이터를 출력하는 방식
    - Fluentd
        - 오픈소스 데이터 수집 도구
        - 카프카에 특화된 것은 아니지만, 각각 입출력용의 플러그인이 제공된다.
        - 프로듀서쪽에서도 사용할 수 있고 컨슈머 쪽 도구로도 연결할 수 있다.
    - Kafka Rest Proxy
        - confluent가 오픈소스로 개발하고 있는 RESTFul API다.
        - 이걸 사용하면 http프로토콜로 각종 조작이 가능하다.
        - 카프카 클러스터와 별도로 데몬으로 띄운다.
        - 보안상 이유로 외부 시스템과의 방화벽 제한을 하고 싶다면, 이 서비스를 통해서 kafka와 통신할 수 있다.
    - Apache Flink or Spark
        - 스파크처럼 병렬 분산 스트림 처리가 가능하다
        - 이러한 미들웨어는 카프카에서 직접 데이터를 취득해서 스트림처리와 배치 처리를 모두 지원한다
        - Spark
            - oss의 병렬 분산 처리 프레임워크다
            - 맵리듀스보다 더 효율적이다
            - 데이터 처리 모델
                - 처리 대상 데이터를 RDD(Resilient Distributed Dataset)
    - Kafka Streams
        - 카프카가 빌트인으로 제공하는 스트림 처리를 위한 API다.
        - Stream DSL이라는 추상도가 높은 API 일부
        - 대표적으로 Apache Storm, Apache Flink, Spark Streaming

## 참고 코드

- Kafka Connect
    - ByteArrayConverter

## 고민해볼 만한 부분

- 링크드인은 엄격한 트랜잭션 관리는 다소 오버스펙이며, 높은 처리량 실현이 우선순위가 더 높았었다. 메시지 분실 없이 송수신 보증을 너무 중시하다보니 처리량이 나오지 않는 상황에서 카프카 출현했다고 하는데..
- push하는 것보다, 수신시스템이 메시지를 pull하는 방식