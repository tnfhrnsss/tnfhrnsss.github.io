---
layout: post
title: Feign Retry
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Home
# parent: Feign
# grand_parent: Msa
nav_exclude: true
---

# feign retry

1. 전역의 ribbon설정을 안쓰고자 함
2. docker의 depends on도 방법

healthcheck:
test: ["CMD", "curl", "-f", "[http://localhost:8040/actuator/health](http://localhost:8040/actuator/health)"]
interval: 10s
timeout: 5s
retries: 10

3. feignclient 내부 configuration으로 Retryer를 Bean으로 등록해서 backoff제어한다.

참고) 

[우아한 feign 적용기 - 우아한형제들 기술 블로그](https://woowabros.github.io/experience/2019/05/29/feign.html#%EC%A2%80%EB%8D%94-%EB%82%98%EC%95%84%EA%B0%80%EA%B8%B0)