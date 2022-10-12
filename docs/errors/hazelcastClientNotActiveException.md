---
layout: post
title: HazelcastClientNotActiveException
date: 2022-03-15 21:00:00
last_modified_at : 2022-03-15 21:00:00
parent: Errors
has_children: false
nav_exclude: true
tags: [hazelcast]
---

## error

```prolog
HazelcastClientNotActiveException: Client is shutting down
```

```prolog
com.hazelcast.client.HazelcastClientNotActiveException: Client is not active.
at com.hazelcast.client.impl.clientside.HazelcastClientProxy.getClient(HazelcastClientProxy.java:303)
at com.hazelcast.client.impl.clientside.HazelcastClientProxy.executeTransaction(HazelcastClientProxy.java:173)
at org.springframework.boot.actuate.hazelcast.HazelcastHealthIndicator.doHealthCheck(HazelcastHealthIndicator.java:49)
at org.springframework.boot.actuate.health.AbstractHealthIndicator.health(AbstractHealthIndicator.java:82)
at org.springframework.boot.actuate.health.HealthIndicator.getHealth(HealthIndicator.java:37)
at org.springframework.boot.actuate.health.HealthEndpointWebExtension.getHealth(HealthEndpointWebExtension.java:85)
at org.springframework.boot.actuate.health.HealthEndpointWebExtension.getHealth(HealthEndpointWebExtension.java:44)
at org.springframework.boot.actuate.health.HealthEndpointSupport.getContribution(HealthEndpointSupport.java:99)
at org.springframework.boot.actuate.health.HealthEndpointSupport.getAggregateHealth(HealthEndpointSupport.java:110)
at org.springframework.boot.actuate.health.HealthEndpointSupport.getContribution(HealthEndpointSupport.java:96)
at org.springframework.boot.actuate.health.HealthEndpointSupport.getHealth(HealthEndpointSupport.java:74)
at org.springframework.boot.actuate.health.HealthEndpointSupport.getHealth(HealthEndpointSupport.java:61)
at org.springframework.boot.actuate.health.HealthEndpointWebExtension.health(HealthEndpointWebExtension.java:71)
at org.springframework.boot.actuate.health.HealthEndpointWebExtension.health(HealthEndpointWebExtension.java:60)
at sun.reflect.GeneratedMethodAccessor573.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
```

## cause

hazelcast를 사용하는 어플리케이션이 동일 clientId로 서비스를 여러 개 올릴 경우 발생하는 에러

## solve

1. 서비스가 이미 떠있는 상태는 아닌지 프로세스 체크
2. 실행 중인 프로세스가 없는 경우, eureka에 아직 UP상태 또는 SHUT DOWN중인 상태가 아닌지 체크
3. 의존성을 갖는 서비스에서 리소스를 아직 잡고 있는 것은 아닌지 체크