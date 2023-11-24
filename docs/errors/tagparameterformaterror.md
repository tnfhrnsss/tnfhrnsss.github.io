---
layout: post
title: Each tag parameter must be in the form 'key:value' InvalidEndpointRequestException
date: 2023-11-24 23:51:00
last_modified_at : 2023-11-24 23:51:00
parent: Errors
has_children: false
nav_exclude: true
description: Each tag parameter must be in the form 'key:value'
---

# error log

```prolog
"Each tag parameter must be in the form 'key:value'"; 
nested exception is org.springframework.boot.actuate.endpoint.
InvalidEndpointRequestException: 

Each tag parameter must be in the form 'key:value' 
but was: test-data-exists,
```

# cause

캐시 annotation에 사용되는 값들은 태그 형식? spel과 같은 형태로 key:value에 맞춰서 작성해야하는데 그게 맞지 않아서 발생한 것으로

spring boot actuator에서 이러한 태그들 포맷에 대해 검사를 하다보니 약간은 엉뚱?할수 있는 InvalidEndpointRequestException이 발생해서  원인파악하기 쉽지 않다.

# problem

```java
@CacheEvict(
value = {"test-data-find, test-data-exists"}, key = "#userId")
```

여러 개의 캐시를 선언할 땐 쌍따옴표로 구분해야한다.

# solved

```java
@CacheEvict(value = {"test-data-find", "test-data-exists"}, key = "#userId")
```