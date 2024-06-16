---
layout: post
title: `@DynamicUpdate에 대해서 캔버스톡
date: 2024-06-16 22:07:00
last_modified_at : 2024-06-16 22:07:00
parent: Jpa
grand_parent: Msa
nav_exclude: true
tags: [jpa, hibernate]
---

# 1. @DynamicUpdate는 무엇인가요?

- hibernate-core에서 지원하는 어노테이션
- Target은 ElementType.Type으로 클래스에만 적용
- 주로 엔티티 클래스에 적용되어서 db에 **`동적`**으로 반영하기 위함

# 2. 제품 적용 배경

- 기본적으로 hibernate는 변경여부 상관없이 모든 필드를 save하고 있기 때문에 불필요한 연산이 일어나는 문제가 있어요~!
- 이 점을 보완하기 위한 방법 중 한가지로 @DynamicUpdate를 사용할 수 있습니다.

# 3. 그렇다면 어떤 얘기가 필요할까요?

- 문제는 DynamicUpdate가 적용된 클래스에 컬럼수가 많은 경우는 그만큼 동적쿼리를 생성시켜서 성능저하를 발생시키고 있는데요.
- 성능 저하 지점
    - DynamicUpdate로 설정한 엔티티에 의해 동적으로 생성된 쿼리는 캐싱되지 않아서 db서버에 부하발생
    - hibernate내에 관련 변경 필드에 대해 추적하기 위한 자원 소모?!

![about_dynamicUpdate](../img/about_dynamicUpdate.png)