---
layout: default
title: jpa 프로그래밍
parent: Jpa
has_children: false
nav_order: 1
grand_parent: Clipping
---

## 영속성 관리

#### @DynamicUpdate

컬럼이 30개 이상이 되면, @DynamicUpdate를 사용한 동적 수정 쿼리가 정적 수정 쿼리보다 빠르다. 
하지만 한 테이블에 컬럼이 30개 이상 된다는 것은 테이블 설계상 분리가 적절하게 되지 않았을 가능성이 높다.

### @SequenceGenerator

SequenceGenterator.allocationSize의 기본값은 50이다. 이는 최적화 때문인데, 만약 하나씩 증가해야한다면 1로 설정하면 된다.
@SeqyebceGeberator는 @GeneratedValue 옆에 사용해도 된다.

## 엔티티 맵핑

### @Column 생략

~~~
int data1;  // @Column생략, 자바 기본타입으로 not null로 생성된다.

Integer data2;  // @Column 생략하면 nullable속성으로 생성된다.
~~~

따라서 자바 기본타입에 @Column을 사용하면 nullable = false로 지정하는 것이 안전하다.

## 연관관계 매핑 기초

### 양방향 매핑
양방향 매핑시에는 무한 루프에 빠지지 않게 조심해야한다. 
예를 들어 Member.toString()에서 getTeam()을 호출하고 
Team.toString()에서 getMember()를 호출하면 무한 루프에 빠질 수 있다. 
이런 문제는 엔티티를 JSON으로 변환할 때 자주 발생한다.

## 다양한 연관관계 매핑

### 일대다
- 일대다 단방향 매핑보다는 <ins>다대일 양방향</ins> 매핑을 사용하자.
- 관계형 데이터베이스 특성상 외래키는 항상 다 쪽에 있다. 그러므로 양방향 매핑에서 @OneToMany는 연관관계의 주인이 될 수 없다. 이런 이유로 @ManyToOne에는 mappedBy속성이 없다.

