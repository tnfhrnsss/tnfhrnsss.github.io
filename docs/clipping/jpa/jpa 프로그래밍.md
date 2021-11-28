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




