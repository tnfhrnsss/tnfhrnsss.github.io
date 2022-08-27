---
layout: post
title: 우아한 CRUD - 퍼시스턴스 프레임워크 정리
date: 2022-08-27 21:32:00
last_modified_at : 2022-08-27 21:32:00
parent: Youtube
grand_parent: Mooc
nav_exclude: true
---

# 강의 주소

[[우아한테크세미나] 200507 우아한CRUD by 정상혁&이명현님](https://www.youtube.com/watch?v=cflK7FTGPlg)

## model1

- 우리가 흔히 과거에 했던 jsp에 sql까지 다 있던 코드패턴
- smart ui패턴이라고도 한다(ddd 에릭에반스)

## JPA

- User*Account클래스를 보면 ORM사용 성숙도를 알 수 있다
- 부작용
    - 성능저하
        - 항상 연관된 객체를 다 조회한다면 불필요한 쿼리 실행
        - N+1 쿼리 주의
        - lazy loading을 쓰지않고 수동으로 값을 채울 때의 난관
            - 비슷한 메서드가 여러개 생길 수 있다. (find로 찾는 여러개의 메서드)

## VO vs DTO

- Value Object
    - 값이 같으면 동일하다고 간주되는, 식별성 없는 작은 객체
    - hibernate 매뉴얼의 Value Type도 Value Object를 포함한다고 생각됨
    - DTO와 혼용해서 쓰여있다.
- Data Transfer Object
    - 원격 호출을 효율화하기 위해 나온 패턴

## Entity 감추기

- 이유
    - entity가 뷰, API의 response에 바로 노출될 때의 비용이 발생
        - 캡슐화를 지키기 어려워진다 : 꼭 필요하지 않은 속성도 외부로 노출되어 향후 수정하기 어려워진다.
        - JSP, Freemarker에서의 객체 참조
            - 컴파일 시점의 검사 범위가 좁다
            - jpa를 쓴다면 , OpenEntityManageInViewFilter를 고려해야한다. 예를 들어서 jsp에서 쿼리 호출할 경우
        - JSON 응답시
            - JsonIgnore, JsonView 같은 선언이 많아지면, JSON형태만 보고 예측하는 난이도가 올라간다.
- 해결
    - 외부 노출용 DTO를 따로 만들기
        - entity → dto 변환 로직은 컴파일 타임에 체크된다
        - dto는 비교적 구조를 단순하게 가져갈 수 있다 (더 단순한 json응답 구조로)
        - dto변화는 외부 인터페이스로 의식해서 관리하는 범위가 된다. (예를 들어 swagger의 스펙으로 활용)
        - 여러 entity를 조합할 수 있는 여지가 생긴다.
    

## Aggregate로 Entity 간의 선긋기

- Aggregate는?
    - 하나의 단위로 취급되는 연관된 객체군, 객체망
    - entity와 vo의 묶음
    - 엄격한 데이터 일관성, 제약사항이 유지되어야 하는 단위
    - transaction, lock의 필수 범위
    - 불변식이 적용되는 단위(데이터가 변경될 때마다 유지돼야 하는 규칙)
- Aggregate 1개 당 repository 1개
    - Aggregate root를 통해서 aggregate밖에서 aggregate안의 객체로 접근한다.
    - Aggregate root로 저장 대상 타입을 표현하는 CrudRepository
- Aggregate 경계가 있는 시스템
    - 별도의 저장소나 API서버를 분리할 때 상대적으로 유리
        - **Aggregate밖은 eventual consistancy를 목표로 할 수 있다**
        - **여러 Aggregate의 변경은 Event, SAGA, TCC 등의 패턴을 활용할 수 있다**
    - Aggregate별로 Cache를 적용하기에 좋다
    - 분리할 계획이 없더라도 코드를 고칠 때 영향성을 파악하기가 유리하다.
- Aggregate 식별시 의식할 점
    - CUD + 단순(findById)에 집중
        - 모든 R( = read)를 다 포용하려고 한다면 깊은 객체 그래프가 나온다.
    - JPA를 쓴다면 Cascade를 써도 되는 범위인가??
        - cascade를 꼭 써야한다고 하면, entity의 경계가 잘못된건 아닌지 다시 생각해봐야하는 신호라고 봐야한다!!

## Aggregate 간의 참조

- 다른 Aggregate를 참조할때, 객체로 참조하지 않고 ID로만 참조하도록 한다.
- Spring Data Jdbc의 AggregateReference도 같은 역할

## 여러 Aggregate에 걸쳐 조회

- Service레이어에서 여러개를 조합한다.
- 쿼리 하나로 하면 비용이 적게 든다고 생각할 수 있지만, 여러 아이디를 findBy하면 DB성능에 더 유리할 수 있다. 각각의 쿼리가 더 단순하고 Application DB레벨의 캐시에 더 유리하다
- Join이 필수적인 경우
    - 예를 들어, where절에 다른 aggregate속성이 조건으로 들어가는 경우는
        - Repository에 조회조건 정도(findByCreatorEmail(String email)이렇게 추가하고 Service단에서 다시 조합하는 방법이 있다..
- select 결과까지 다른 aggregate의 속성을 포함할 경우
    - 맞춤형 전용 dto를 만드는 방법이 있다. 이러면
        - Aggregate를 단순하게 유지할 수 있고
        - JPA의 경우: **Persistent Context**를 의식하지 않아도 된다.

## Repository vs DAO

- Dao는 퍼시스턴스 레이어를 캡슐화
- ddd의 repository는 도메인 레이어에 객체 지향적인 컬렉션 관리 인터페이스를 제공
- Lazy Loading이 필요하다는 것은 모델링을 다시 생각해봐야한다는 신호일 수도 있다.

## Immutable과 rich domain object

- 캐시 부작용 사례
    - 캐시된 값을 findbyid로 가져와서, 객체의 상태를 바꿔버리는 것으로 의도치 않게 오류가 발생할 수 있다.
    - 사내에서 발생한 사례로 .. 아래처럼 코딩했을 때 캐시 변경이 일어났다고 한다.
        
        ```java
        public static InternalApprovalReviewCdo from(ApprovalRequest approvalRequest, IdList essentialApproverIds) {
                IdList approverIds = approvalRequest.getApproverIds();
                approverIds.addAll(essentialApproverIds);
                return new InternalApprovalReviewCdo(
                    approverIds,
                    approvalRequest.getCreatedAt(),
                    approvalRequest.getDueDate()
                );
            }
        ```
        
    - from메서드의 인자 approvalRequest는 Repository에서 findBy로 리턴받은 캐시 객체인데, 라인3 에서 addAll하면 approvalRequest 저장된 값이 변경되는 것과 같은 사례인 것 같다.

- immutable 객체의 장점
    - cache하기에 안전하다
    - 다른 레이어에 메서드 파라미터로 보내도 값이 안 바뀌었다는 확신을 할 수 있다.
    - dto류가 여러 레이어를 오간다면 immutable하면 더 좋다.
- Rich Domain Object
    - Domain Object가 가진 속성과 연관된 행위
        - 해당 객체에 있는 것이 책임이 자연스럽다(**information expert패턴**)
        - 데이터 중심에서 책임 중심의 설계로 진화할 수 있다.
    - 상태를 바꾸는 메서드가 포함될 수 있다
        - 상태를 바꿀 때의 정합성 검사를 포함
        - Domain event 추가(Spring Data의 AbstractAggregateRoot)로 해볼 수 있다.
    - Immutable이 아니게 될 수 있다
    - 영속화될 Domain Object라면 상태를 바꾸는건 시스템의 상태를 바꾸는 경우에 한해야한다.
        - 메서드명도 그 행위를 잘 들어내어야 한다(setTitle() → changeTitle())

## 실무 사례

- 복합조회 API를 통해 화면을 만들어야 하는 경우
    - 복잡한 쿼리를 담당하는 API서버 분리 개발 (read를 담당하는)
    - Aggregate경계를 넘어서는 복잡한 조회이기 때문에 Repository가 아닌 Dao개념으로 접근하고
    - Native Sql 중심의 개발로 - 쿼리 힌트, 최적화를 위해 복잡한 형태의 쿼리가 들어가게 됨
    - **Spring jdbc+ Spring data jdbc의 일부 기능 사용**
        - **jpa dialect가 없는 자체 분산 DB(NBase-T)사용**
        - jpa를 쓸 수 있다고 해도 적용의 이득이 없는 상황

## 결론

- Aggregate
- 객체지향 설계는 (j2ee나 자바같은) 특정 구현 기술보다 더 중요하다(로드 존슨)
    - entity 설계는 (jpa나 spring jdbc같은) 특정 구현기술 보다 더 중요하다