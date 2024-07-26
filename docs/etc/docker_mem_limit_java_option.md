---
layout: post
title: docker-compose mem_limit and java_option
description: docker-compose mem_limit and java_option
date: 2024-07-26 13:32:00
last_modified_at : 2024-07-26 13:32:00
parent: Etc
has_children: false
nav_exclude: true
---

# how to to set Java memory option in a `docker-compose.yml`

- JAVA_TOOL_OPTIONS 옵션을 사용해서 추가할 수 있다.

```yaml
cstalk:
  image: xxx./repository/test:${VERSION}
  container_name: test
  environment:
    - "TZ=Asia/Seoul"
    - "SPRING_PROFILES_ACTIVE=local"
    - "JAVA_TOOL_OPTIONS=-Xmx512m -Xms256m"
```

# 그렇다면 mem_limit 옵션은 무엇인가?

- mem_limit은 Docker에 올린 컨테이너가 사용할 수 있는 최대 메모리 크기다.
- mem_limit는 JVM의 힙 메모리에 대한 설정이 아니기 때문에 자바 옵션 관련해서 설정하려고 하면 **JAVA_TOOL_OPTIONS** 또는 **JAVA_OPTS** 환경 변수를 사용한다.
- 이 설정을 해두면
    - 특정 컨테이너가 메모리를 지나치게 점유하는 것을 막을 수 있다.