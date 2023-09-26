---
layout: post
title: SpelEvaluationException EL1008E CacheExpressionRootObject
date: 2022-07-25 21:40:00
last_modified_at : 2022-07-25 21:40:00
parent: Errors
has_children: false
nav_exclude: true
tags: [spring, cache]
---

## error log

```prolog
org.springframework.expression.spel.SpelEvaluationException: EL1008E: Property or field 'license_key' 
cannot be found on object of type 'org.springframework.cache.interceptor.CacheExpressionRootObject' 
- maybe not public or not valid?
at org.springframework.expression.spel.ast.PropertyOrFieldReference.readProperty(PropertyOrFieldReference.java:217)
at org.springframework.expression.spel.ast.PropertyOrFieldReference.getValueInternal(PropertyOrFieldReference.java:104)
at org.springframework.expression.spel.ast.PropertyOrFieldReference.getValueInternal(PropertyOrFieldReference.java:91)
```

문제 코드

```java
@Cacheable(value = "crema-btalk-license-find", key = "license_key", unless = "#result == null")
    public LicenseRdo find() {
```

## cause

key를 string값으로 주고 싶다면 single quote를 줘야한다. 

## solved

```java
@Cacheable(value = "crema-btalk-license-find", key = " 'license_key' ", unless = "#result == null")
    public LicenseRdo find() {
```

아니면, 메소드명과 동일하게 하거나

```java
@Cacheable(key = "#root.methodName")
```

아니면, 전역변수로 선언하거나

- 전역변수를 key로 사용하려면
* 반드시 public해야하고
* 반드시 static final 상태여야 한다.

```java
public static final String KEY = "cacheKey";

@Override
@Cacheable(value = "cacheName", key = "#root.target.KEY")
```

## reference

[What is the best way of defining key for @Cacheable annotation for Spring](https://stackoverflow.com/a/62356500)