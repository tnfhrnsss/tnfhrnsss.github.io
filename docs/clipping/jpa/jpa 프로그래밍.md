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




