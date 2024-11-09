---
layout: post
title: Cannot connect to the Docker daemon at unix
description: Cannot connect to the Docker daemon at unix
date: 2024-11-09 18:38:00
last_modified_at : 2024-11-09 18:38:00
parent: Errors
has_children: false
nav_exclude: true
tags: [docker]
keywords: docker
---


# Problem

- intellij를 통해 docker-compose.yml를 실행하는 중 에러 발생.
- 늘 이미지를 삭제하고 인텔리제이를 통해 서비스를 올려왔었는데 갑자기 발생한 에러..
    - 의심가는 상황은.. 시작프로그램에서 docker를  재시작했었는데 이 때문인가 🤔

# Error

```prolog
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. 
Is the docker daemon running? (Details: [2] No such file or directory)
```

# Solved??

- 그냥 도커 버전 업그레이드하면서 자동 치유… 😆