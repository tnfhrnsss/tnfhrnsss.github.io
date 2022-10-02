---
layout: post
title: spring boot and cloud version up
date: 2022-10-01 09:48:00
last_modified_at : 2022-10-01 09:48:00
parent: SpringCloud
grand_parent: Msa
nav_exclude: true
tags: [spring cloud, spring boot]
---

# spring 업그레이드 대응개발

# summary

- 버전 변경
    - Spring Boot : 2.3.12.RELEASE -> 2.7.3
    - Spring Cloud : Hoxton.SR12 -> 2021.03
- 주요 작업
    - Hystrix, Ribbon 이 기존 Spring Clou Netflix 에서 지원되지 않고, 다른 스택으로 대체됨에 따라 기존에 의존하고 있던 코드의 변경 및 설정 변경작업

# 작업 내용

[change netflix-ribbon to spring-cloud-loadbalancer](./docs/msa/spring-cloud/spring_upgrade_scl.md)  
[change Hystrix to Resilience4j](./docs/msa/spring-cloud/spring_upgrade_resilience4j.md)  
[change Feign And Loadbalancer Retry](./docs/msa/spring-cloud/spring_upgrade_retry.md)  


# reference

[https://sabarada.tistory.com/204](https://sabarada.tistory.com/204)

[https://luvstudy.tistory.com/150](https://luvstudy.tistory.com/150)