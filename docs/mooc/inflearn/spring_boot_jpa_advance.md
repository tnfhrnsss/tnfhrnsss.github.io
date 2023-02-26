---
layout: post
title: 실전! 스프링 부트와 JPA활용2
date: 2023-02-26 14:36:00
last_modified_at : 2023-02-26 14:36:00
parent: Inflearn
grand_parent: Mooc
nav_exclude: true
tags: [spring]
---

# 강의 소개

- 인프런, 김영한님
- 총 24강 395분
- 실무에 도움되는 것들로 구성되어있다. 극약처방 같은 느낌

# Summary

## API 개발 기본

### 1. API 스펙이랑 Entity가 1:1로 맵핑되어서 발생하는 문제

1. 즉, Entity가 변경되면, API스펙도 변경이 수반되는 문제
2. 조치
    1. Dto객체를 생성해서 Entity를 대신하도록 한다.
    2. 즉, 프레젠테이션 계층을 위한 Result 클래스를 생성한다
3. API스펙에 다 노출하지 말고 필요한 것만 노출하도록 한다.

## API 개발 고급

1. 지연 로딩과 조회 성능 최적화
    1. API 스펙으로 엔티티를 그대로 리턴하면 발생하는 문제
        1. 엔티티가 fetch=LAZY일 때
            1. proxy를 통해 ByteBuddyInterceptor 발생하니까 객체 강제 초기화를 하거나 hibernate모듈 디펜던시 걸거나
            2. 지연로딩을 피하기 위해서 즉시로딩(eagerLoading)을 사용하면 안된다.
                1. 즉시로딩 때문에 연관관계가 필요 없는 경우에도 데이터를 항상 조회해서 성능 문제가 발생할 수 있다. 튜닝하기도 어려워진다.
                2. 항상 지연 로딩을 기본으로 하고, 성능 최적화가 필요한 경우에는 패치 조인(fetch join)을 사용하도록 한다.
        2. 객체간 양방향 연관관계가 있을 때 (Member와 Order객체가 서로 참조할 때)
            1. 한 곳을 jsonIgnore를 선언해줘야한다.
    2. store에서 조회한 값을 도메인 객체로 변환하는 과정에서 발생하는 N번 쿼링
        
        ```java
        @Data
        static class SimpleOrderDto {
        
        	public SimpleOrderDto(Order order) {
        		orderId = order.getId();
        		name = order.getMember().getName();
        		orderDate = order.getOrderDate();
        		orderStatus = order.getStatus();
        		address = order.getDelivery().getAddress();
        	}
        }
        ```
        
        1. 쿼리가 총 1+N+N번 실행된다.
            1. order조회 
            2. order → member 지연 로딩 조회 N번
            3. order → delivery 지연 로딩 조회 N번
        2. 조치방법
            1. lazy로딩으로 변경한다.
                1. 지연로딩은 영속성 컨텍스트에서 조회하므로, 이미 조회된 경우 쿼리를 생략한다.
            2. 패치조인으로 최적화한다.
                
                ```java
                public List<Order> findAllWithMemberDelivery() {
                	return em.createQuery(
                			"select o from Order o" + 
                			" join fetch o.member m" +
                			" join fetch o.delivery d",
                			Order.class
                	).getResultList();
                }
                ```
                
                1. 엔티티를 fetch join을 사용해서 쿼리 1번에 조회
                2. 패치 조인을 order → member, order → delivery는 이미 조회된 상태이므로 지연로딩자체가 일어나지 않는다.
            3. 웬만한 jpa성능은 lazy 로딩 + 패치 조인으로 다 해결된다
    3. JPA에서 DTO로 바로 조회
        1. 화면에서 호출이 많은 조회용 API인 경우, select 절을 통해 가져올 데이터만 가져오게 해서 dto로 바로 조회하는 방법을 사용하면 네트워크를 줄일 수 있다.
    4. 그래도 개선이 안된다면
        1. JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사용한다.

### 2. 컬렉션 조회 최적화 (젤 중요‼️)

1. 패치 조인 최적화
    1. 패치 조인시 1:다 조인이므로 중복 데이터는 distinct를 통해 리턴하도록 한다.
        1. 단점은 페이징 불가
    2. 컬렉션 패치 조인은 1개만 사용할 수 있다. 컬렉션 둘 이상에 패치 조인을 사용하면 안된다. 페이지가 부정합하게 조회될 수 있다.
2. 페이징 + 컬렉션 엔티티 조인 문제 해결 방법
    1. ToOne(OneToOne, ManyToOne)관계를 모두 패치조인한다. ToOne관계를 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지않는다.
    2. 컬렉션은 지연 로딩으로 조회한다.
        1. 지연 로딩 성능 최적화를 위해 hibernate.default_batch_fetch_size(전역 설정), @BatchSize(개별 설정)를 적용한다
            1. 이 옵션을 사용하면 컬렉션이나 프록시 객체를 한꺼번에 설정한 size만큼 IN쿼리로 조회한다.
            2. 웬만하면 이 옵션을 통해서 성능 최적화가능하다. 데이터양이 많으면 패치조인보다 낫다.
            3. 쿼리 호출수가 1 +N에서 1+1로 최적화된다.
            4. 웬만하면 100~1000사이를 선택하도록 한다. IN절 파라미터가 1000개로 제한이 있기도 하고 1000개를 한번에 불러오면 순간 DB부하가 올라갈 수 있기 때문에 그러함
3. JPA에서 DTO 직접 조회
    
    ```java
    public List<OrderQueryDto> findOrderQueryDots() {
    	return em.createQuery(
    			"select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, om.name)" +
          " from Order o" +
          " join o.member m" +
          " join o.delivery d", OrderQueryDto.class)
          .getResultList(); 
    
    }
    ```
    

### 3. 정리

1. 엔티티 조회 방식으로 우선 접근
    1. 패치 조인으로 쿼리 수를 최적화
    2. 컬렉션 최적화
        1. 페이징이 필요한 경우 fetchsize 옵션 사용으로 최적화
        2. 페이징이 필요하지 않은 경우 → 패치 조인 사용
    3. 엔티티 조회방식으로 해결이 안되면 DTO조회 방식 사용
    4. DTO조회 방식으로 해결이 안되면 NativeSQL or 스프링 JdbcTemplate을 사용한다.
2. 캐시는 Entity에 캐시하면 안된다. 영속성 컨텍스트때문에 Entity에 캐시하게 되면 복잡해진다. 캐시는 Dto에 해야한다.

## OSIV와 성능 최적화

- open session in view : 하이버네이트 (jpa가 없었을 때)
- open entitymanager in view : jpa 로 되나, 관례상 osiv라 한다
- 설정
    
    ```yaml
    spring.jpa.open-in-view: true (default)
    ```
    
    - 서비스단에서 @Transactional 을 보통 선언해두고, 서비스계층에서 트랜잭션이 시작할 때 db connection을 가져오는데, api가 user에게 반환될 때까지 트랜잭션이 종료해도 영속성 컨텍스트는 살아 있다.  화면의 경우는 렌더링이 완료될 때까지 데이터 커넥션을 물고 있다.
- open session in view를 false로 하면 커넥션을 빨리 반환해주기 때문에 커넥션 리소스를 낭비하지 않는다. 다만
    - osiv를 끄면
        - 모든 지연로딩을 트랜잭션 안에서 처리해야 한다는 것
        - view template에서 지연로딩이 동작하지 않는다는 것
        - 결론적으로 트랜잭션이 끝나기 전에 지연 로딩을 강제로 호출해 두어야 한다.
- osiv를 false로 하고 이렇게 구성한다
    - 커맨드와 쿼리를 분리한다.
    - 예로 OrderService
        - OrderService : 핵심 비즈니스 로직
        - OrderQueryService: 화면이나 API에 맞춘 서비스(주로 읽기 전용 트랜잭션 사용하는 경우)