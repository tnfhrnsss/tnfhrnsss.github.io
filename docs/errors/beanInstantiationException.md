---
layout: post
title: BeanInstantiationException Failed to instantiate
date: 2022-09-07 21:41:00
last_modified_at : 2022-09-07 21:41:00
parent: Errors
has_children: false
nav_exclude: true
tags: [spring boot]
---

## problem

```prolog
Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: 
Error creating bean with name 'org.springframework.security.config.annotation.web.reactive.WebFluxSecurityConfiguration': Unsatisfied dependency expressed through method 'setSecurityWebFilterChains' parameter 0; 
nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'filterChain' defined in class path resource [WebFluxSecurityConfig.class]: 
Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.security.web.server.SecurityWebFilterChain]:
Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.security.web.server.SecurityWebFilterChain]: 
Factory method 'filterChain' threw exception; nested exception is java.lang.NullPointerException
	
Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.security.web.server.SecurityWebFilterChain]: Factory method 'filterChain' threw exception; nested exception is java.lang.NullPointerException
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:185)
	at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:653)
	... 36 common frames omitted
Caused by: java.lang.NullPointerException: null
		at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:154)
	... 37 common frames omitted
```

현상은 springboot 서비스 run할 때, 위의 에러를 내뱉고 죽었다.

run할 때 발생하는 에러에서 npe나 bean create못한다는 에러이면, yml설정을 의심해보는 것이 좋다

## cause

위의 경우는

SecurityWebFilterChain을 bean으로 등록하는 클래스에 아래의 코드가 있었는데

```prolog
@Value("${security.ignores:}")
private String[] securityIgnores;
```

- property를 설정한 yml을 읽어오지 못해서 npe가 발생하고 bean등록에 실패한 것이였다
- yml을 못가져온 것은 application.yml의 search-locations 끝에 / 슬래시가 없어서 (window는 끝에 슬래시 필수)

## reference

[Error creating bean with name 'springSecurityFilterChain'](https://stackoverflow.com/a/28655714)