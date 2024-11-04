---
layout: post
title: A bean with that name has already been defined and overriding is disabled.
date: 2024-11-04 09:14:00
last_modified_at : 2024-11-04 09:14:00
parent: Errors
has_children: false
nav_exclude: true
tags: [feign]
description: A bean with that name has already been defined and overriding is disabled.
---


# Error

```log
***************************
APPLICATION FAILED TO START
***************************

Description:

The bean 'xxx.xxx' could not be registered. 
A bean with that name has already been defined and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by 
setting spring.main.allow-bean-definition-overriding=true
```

# Problem

```java

@FeignClient(
    contextId = "xxx.xxx",
    name = "test",
    configuration = FeignConfiguration.class,
    primary = false
)
```

# Cause

- Feign Client는 기본적으로 contextId값으로 다른 클라이언트와 구분하기도 하고 빈으로 등록되는 값을 사용된다.
- 만약에 contextId가 빈값이면 feign의 기본값으로 설정되고, 다른 클래스에서도 빈값으로 두면 중복되어 위의 에러가 발생하기도 하고
- contextId를 정의했다 하더라도 다른 클래스의 contextId와 동일한 값이면 위의 에러가 발생한다.

# Solve

- 중복되지 않도록 contextId를 package + class name 구성으로 해주는 것이 좋다.