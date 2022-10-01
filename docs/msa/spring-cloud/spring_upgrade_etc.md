---
layout: post
title: spring boot and cloud version up - etc
date: 2022-10-01 09:48:00
last_modified_at : 2022-10-01 09:48:00
parent: SpringCloud
grand_parent: Msa
nav_exclude: true
tags: [spring cloud, spring boot]
---

breaking changes를 보면

[https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2020.0-Release-Notes#breaking-changes-1](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2020.0-Release-Notes#breaking-changes-1)

# Actuator

- endpoint에서 dash가 사라졌다
1. bus-refresh가 busrefresh로 바뀌었나는 것
2. bus-env도 busenv로 바뀌었다는 것
3. service-registry도 serviceregistry로 

```prolog
curl -X POST http://localhost:8888/actuator/busrefresh
```