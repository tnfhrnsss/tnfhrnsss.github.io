---
layout: post
title: process leaked file descriptors
description: process leaked file descriptors
date: 2023-07-27 22:11:00
last_modified_at : 2023-07-27 22:11:00
parent: Errors
has_children: false
nav_exclude: true
tags: [jenkins]
---

# problem

jenkins로 빌드 후 작업에서 execute shell로 spring application을 실행시키는 sh을 호출하는 작업을 설정했는데

“process leaked file descriptors” 라고 로깅되고 서비스가 내려가 있었다.

# cause

원인은 jenkins job이 다 수행되면, 백그라운드로 실행되도록 sh을 작성해도 프로세스를 종료시킨다고 한다.

[https://stackoverflow.com/a/45859181/14257397](https://stackoverflow.com/a/45859181/14257397) 

# solved

jenkins 관리 > 시스템 변수 설정화면에서 변수를 추가한다.

- key : BUILD_ID
- value : allow_to_run_as_daemon start_my_service

![process_leaked_file_descriptors](../img/process_leaked_file_descriptors.png)