---
layout: post
title: 실전 Querydsl 수강 정리
date: 2022-10-01 22:11:00
last_modified_at : 2022-10-15 23:52:00
parent: Inflearn
grand_parent: Mooc
nav_exclude: true
tags: [querydsl, jpa]
---

# 강의 주소

[실전! Querydsl](https://inf.run/Zdvf){:target="_blank"}

# 소개

- 문법 오류를 컴파일 시점에 잡아준다
- 동적 쿼리 문제 해결해준다
- 쉬운 sql스타일 문법

# 환경 설정

- 프로젝트 생성
    - dependency: web, jpa, h2, lombok
- Querydsl 설정과 검증
    - querydsl-jpa 추가
    
    ```yaml
    implementation 'com.querydsl:querydsl-jpa'
    ```
    
    - Q파일 생성되도록 테스트 코드 작성
        - Q파일은 git에 올리면 안된다.(빌드 폴더자체를 ignore시키면 됨)
- h2 설치
    
    ```yaml
    로컬 서비스 띄윅
    /d/programFiles/H2/bin 하위에서 ./h2.bat실행
    ```
    
- 쿼리 로깅
    
    ```java
    spring:
      jpa:    
        properties:          
          show_sql: true        
          format_sql: true        
          use_sql_comments: true
    ```
    

# 기본 문법

1. jpql vs querydsl
    - querydsl은 파라미터 바인딩을 자동으로 한다.
    - jpql은 런타임에 오류가 난다.
2. Q-Type활용
    1. Q클래스 인스턴스를 사용하는 방법 2가지
        1. new생성자에 별칭을 지정해서 사용
            
            ```java
            QMember qMember = new QMember("m");
            ```
            
        2. 기본 인스턴스를 사용하는 방법
            
            ```java
            QMember qMember = QMember.member;e
            ```
            
3. 검색 조건 쿼리
    1. 리스트는 fetch()
    2. fetchOne
    3. fetchFirst : limit 1
    4. fetchResults : 페이징 정보 포함됨. 페이징 쿼리가 복잡해지면, 이걸 쓰지말고 다른 내용 가져오는 쿼리랑 나눠서 두번 호출하는 방식으로 해라. 성능에 안좋다.
    5. fetchCount
4. 집합 함수
    
    ```java
    List<Tuple> result = queryFactory    
    		.select(
    			member.count(),        
    			member.age.sum(),        
    			member.age.avg(),        
    			member.age.max(),        
    			member.age.min())    
    		.from(member)    
    		.fetch();
    ```
    
    - count, sum.. 등등 select결과 단위가 다를 때 Tuple로 리턴받는다.
    - 실무에선 Tuple말고 Dto로 쓴다.
5. 조인
    1. on절
        1. jpa2.1부터 지원
        2. 전혀 상관없는 테이블 조인(세타조인)
        
        ```java
        List<Tuple> result = queryFactory    
        		.select(member, team)    
        		.from(member)    
        		.leftJoin(team) // member.team, team으로 조건을 거는 것과 다름
        		.on(member.username.eq(team.name))    
        		.fetch();
        ```
        
    2. 페치 조인
        1. 성능 최적화일 때 사용한다
        2. 일반 조인과 다르다
            - 일반 Join : join건 entity만 조회
            - Fetch Join : join외에 fetch join된 테이블도 조회
        
        ```java
        쿼리를 보면 분명 join을 했는데 각 Team의 Lazy Entity인 memebers가 
        아직 초기화되지 않았다는 상태를 보여줍니다.
        
        실제로 일반 join은 실제 쿼리에 join을 걸어주기는 하지만 join대상에 대한 영속성까지는 
        관여하지 않습니다.
        
        오직 join만 걸고 실제 영속성 컨텍스트에는 SELECT 대상만을 담게 됩니다.
        위 내용을 기반으로 해서 다시 생각해보면 아래와 같은 정황으로 
        LazyInitializationException이 발생하게 된 것입니다.
        
        이럴 때 .fetchJoin()조건을 걸어서 N + 1조회를 하게 하는 것
        ```
        
        - fetch join을 써서 영속성 컨텍스트에 올려놓기 보다, lazy loading시키는 것
            
            ```java
            @ManyToOne(fetch = FetchType.LAZY)
            @JoinColumn(name = "team_id")
            private Team team;
            ```
            
6. 서브쿼리
    1. jpa, jpql, querydsl 에서의 서브쿼리는 from절(인라인뷰)에서 지원하지 않는다.
        1. 해결방안
            1. 서브쿼리를 join으로 바꿔서 한다
            2. 애플리케이션에서 쿼리를 2번 나눠서 실행시킨다
            3. nativesql을 사용한다
    2. 그러나 하이버네이트 구현체를 사용하면 select절에서 가능하다
    
7. 상수, 문자 더하기
    1. stringValue()를 활용한다. enum을 처리할 떄도 사용한다.

# 중급 문법

1. 프로젝션과 결과 반환
    1. 프로젝션이란 select절에 조회할 값이 먼지 지정하는 것을 말한다.
    2. 프로젝션 대상이 하나이면 타입을 명확하게 지정해서 하고, 둘 이상이면 튜플이나 dto로 조회한다.
        1. Tuple을 repository영역안에서 사용하는 것은 괜찮지만, 서비스영역이나 다른 계층에서 의존을 갖게 되는것은 좋지않은 구조이다.
    3. 순수 jpa에선 new를 통해 결과를 리턴받는 방법
        1. 생성자 방식만 가능하다는 단점
        2. package path를 다 적어줘야 하는 단점
        
        ```java
        List<MemberDto> result = 
        em.createQuery("select new study.querydsl.dto.MemberDto(m.username, m.age) 
        from Member m", MemberDto.class).getResultList();
        ```
        
    
    d. Projections.bean을 통해 setter를 활용하는 방법
    
    e. Projections.field를 활용하는 방법
    
    - getter/setter가 없어도 된다.
    - dto 필드명과 다를 경우, as로 지정해서 fetch해온다.
        - 일치하는 필드가 없을 경우, null로 조회된다.
    
    f. Projections.constructor를 활용하는 방식
    
    - 오류가 있으면 runtime시에 발생하는 단점이 있다
    
    g. @QueryProjection 사용
    
    - 선언하고 compileQuerydsl을 실행시켜야 한다.
    - 그러면 Q파일이 Dto로 생성된다.
        - 컴파일 시점에 오류 확인이 가능한 장점
    - querydsl에 의존성을 갖게 되는 구조는 안되게 작업해야한다.
2. 동적쿼리
    1. BooleanBuilder사용
    2. Where 다중 파라미터 사용
        1. where 조건이 null이면 조건이 무시된다.
3. 수정, 삭제 배치 쿼리
    1. bulk연산
    2. db에 커밋한 정보와 영속성 컨텍스트에 저장되어 있는 정보가 달라진다. 그래서 플러시 해줘야함 (db를 조회한다고 해도 영속성 컨텍스트에 이미 있는 정보는 갱신하지 않기 때문에)
        
        ```java
        em.flush();
        em.clear();
        ```
        

## 순수 JPA리포지토리와 Querydsl

- EntityManager는 proxy로 호출되는 것이기 때문에 동시성 문제는 되지 않는다.
- where절에 파라미터로 조건을 주는 방식은 메서드 단위로 재사용이 가능하게 구성할 수 있다.

# 스프링 데이터 JPA와 Querydsl

## 1. 사용자 정의 리포지토리

- custom 리포지토리용 인터페이스 생성
    
    ```java
    public interface MemberRepositoryCustom {
        List<MemberTeamDto> search(MemberSearchCondition condition);
    }
    ```
    
- 기본 리포지토리 인터페이스에 추가 상속 선언
    
    ```java
    public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
        List<Member> findByUsername(String username);
    }
    ```
    
- 상세 구현 클래스
    
    ```java
    @RequiredArgsConstructor
    public class MemberRepositoryImpl implements MemberRepositoryCustom{
        private final JPAQueryFactory queryFactory;
    
        @Override
        public List<MemberTeamDto> search(MemberSearchCondition condition) {
    			return queryFactory.select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name.as("teamName")
                ))
                .from(member)
                .leftJoin(member.team, team)
                .where(
                    usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe())
                )
                .fetch();
    		}
    }
    ```
    

## 2. Querydsl 페이지 연동

- pageable적용
    
    ```java
    public Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable) {
        QueryResults<MemberTeamDto> results = queryFactory
            .select(new QMemberTeamDto(
                member.id.as("memberId"),
                member.username,
                member.age,
                team.id.as("teamId"),
                team.name.as("teamName")
            ))
            .from(member)
            .where(
                usernameEq(condition.getUsername())
            )
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetchResults();
    
        List<MemberTeamDto> content = results.getResults();
        long total = results.getTotal();
        return new PageImpl<>(content, pageable, total);
    }
    ```
    
- fetchResults를 통해서 count조회까지 호출시켜 2번 호출을 통해 count까지 얻는 방법이 있고, 각각 contents따로 fetchCount따로 호출해서 가져오는 방법이 있는데, 두개의 차이는
    - count쿼리에서는 contents쿼리와 다르게 단순한 케이스가 있다. 예를 들어서 카운트에서는 조인이 필요하지 않는 경우가 있다.
    - count쿼리에 최적화가 필요할 때
- countQuery 최적화
    - count쿼리가 필요없는 경우에 활용하는 방법 (예를 들어서 페이징 단위보다 토탈 페이지수가 작은 경우)
        
        ```java
        // 리턴을 PageImpl이 아니라
        return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchCount);
        ```
        
        - PageableExecutionUtils를 쓰면, 전체 카운트를 체크해 카운트 쿼리가 필요할 때만 호출하게 된다. 카운트 쿼리는 .fetchCount()가 실행되어야 가져온다.
- 스프링 데이터 정렬
    - 스프링 데이터 JPA는 OrderSpecifier로 쉽게 하는데, Querydsl의 정렬로 전환하려면 복잡하기 때문에 파라미터를 받아서 처리하는 것을 권장한다.

# 스프링 데이터 JPA가 제공하는 Querydsl기능

- 인터페이스 제원 - QuerydslPredicateExecutor
    - 단점은 repository를 호출할 때, 서비스나 컨트롤러에서 querydsl 구현 기술에 의존관계를 갖게 된다.
        - 파라미터를 pojo가 아니라 querydsl의 파라미터로 넘겨야한다는 것
    - 실무에서는 한계가 있어서 권장하지 않는다.
- Querydsl Web 지원
    - 컨트롤러에 querydsl을 의존하는 것
    - 단순한 조건만 가능
    - 실무에서 사용하기에 한계가 있음
- 리포지토리 지원 - QuerydslRepositorySupport
    - 편리하긴 하지만, sort는 없음
- Querydsl지원 클래스 만들기
    - 스프링 데이터가 제공하는 페이징을 편리하게 변환
    - 페이징과 카운트 쿼리 분리

## 수강 후기

- 강의가 비교적 짧게 구성되어 있어서 지루하지 않고 수월하게 들을 수 있었습니다.
- querydsl을 적용하게 되면, 많은 참고가 될 것 같습니다.
- 실무에 적용한 경험을 토대로 설명해주셔서 시행착오를 줄일 수 있을 것 같습니다.