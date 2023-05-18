---
layout: post
title: '@Mock vs @MockBean'
date: 2023-05-17 08:07:00
last_modified_at : 2023-05-17 08:07:00
parent: Java
grand_parent: Msa
has_children: false
nav_exclude: true
---

- 공통점
    - 두개 모두 Mockito를 사용해서 테스트코드에서 mock 객체를 생성할 때 사용하는 annotation

## @Mock

- 일반적으로 Spring Test Framework에서 사용된다.
- Mockto.mock()으로도 호출되는 것과 같은 기능이지만, 다른점은 @Mock은 테스트 클래스에서만 사용할 수 있다.
- MockitoAnnotations.initMocks(this)를 호출해서 해당 클래스에서 어노테이션을 처리
- @RunWith(MockitoJunitRunner.class)와 사용

## @MockBean

- Spring Boot Test에서 사용된다.
- MockBean을 통해 스프링 어플리케이션 컨텍스트 내의 bean으로써 mock객체를 등록할 수 있다. 만약에 컨텍스트에 해당 bean이 없다면 새로 생성한다.
- 그래서 실제 bean을 대체하고 외부 서비스나 db에 대한 종속성을 제어할 때 사용한다.
- @RunWith(SpringRunner.class)와 사용