---
layout: post
title: bad class file class file has wrong version 61.0 should be 52.0
description: bad class file class file has wrong version 61.0 should be 52.0
date: 2024-01-07 00:35:00
last_modified_at : 2024-01-07 00:35:00
parent: Errors
has_children: false
nav_exclude: true
---

# Problem

- spring-web을 6.x로 올리고 maven install 또는 build, compile 중 발생

# Error

```java
cannot access org.springframework.web.bind.annotation.PostMapping

bad class file: /org/springframework/spring-web/6.0.6/spring-web-6.0.6.jar
(org/springframework/web/bind/annotation/PostMapping.class)
    class file has wrong version 61.0, should be 52.0
    Please remove or make sure it appears in the correct subdirectory of the classpath.
```

# Cause

- java8로 되어있어서 문제..

# Solution

- java를 17 + 로 올린다.