---
layout: post
title: linux jenkins start failure
date: 2022-10-11 21:56:00
last_modified_at : 2022-10-11 21:56:00
parent: Errors
has_children: false
nav_exclude: true
tags: [jenkins]
---

## problem

linux 서버에 jenkins 설치 후 서비스 올리면 에러 발생

668 ExecStart=/usr/bin/jenkins (code=exited, status=1/FAILURE)

## error

```prolog
# systemctl status jenkins.service
● jenkins.service - Jenkins Continuous Integration Server
   Loaded: loaded (/usr/lib/systemd/system/jenkins.service; disabled; vendor preset: disabled)
   Active: failed (Result: start-limit) since 화 2022-10-11 14:17:14 KST; 5s ago
  Process: 668 ExecStart=/usr/bin/jenkins (code=exited, status=1/FAILURE)
 Main PID: 668 (code=exited, status=1/FAILURE)

10월 11 14:17:14 dev-83 systemd[1]: jenkins.service: main process exited, code=exited, status=1/FAILURE
10월 11 14:17:14 dev-83 systemd[1]: Failed to start Jenkins Continuous Integration Server.
10월 11 14:17:14 dev-83 systemd[1]: Unit jenkins.service entered failed state.
10월 11 14:17:14 dev-83 systemd[1]: jenkins.service failed.
10월 11 14:17:14 dev-83 systemd[1]: jenkins.service holdoff time over, scheduling restart.
10월 11 14:17:14 dev-83 systemd[1]: Stopped Jenkins Continuous Integration Server.
10월 11 14:17:14 dev-83 systemd[1]: start request repeated too quickly for jenkins.service
10월 11 14:17:14 dev-83 systemd[1]: Failed to start Jenkins Continuous Integration Server.
10월 11 14:17:14 dev-83 systemd[1]: Unit jenkins.service entered failed state.
10월 11 14:17:14 dev-83 systemd[1]: jenkins.service failed.
Warning: jenkins.service changed on disk. Run 'systemctl daemon-reload' to reload units.
```

```prolog
journalctl -xe | more 
```

에러 로그를 상세로그로 확인합니다.

## cause

```prolog
10월 11 15:10:24 dev-83 jenkins[4307]: jenkins: invalid Java version: openjdk version "1.8.0_322"
10월 11 15:10:24 dev-83 jenkins[4307]: OpenJDK Runtime Environment (build 1.8.0_322-b06)
10월 11 15:10:24 dev-83 jenkins[4307]: OpenJDK 64-Bit Server VM (build 25.322-b06, mixed mode)
10월 11 15:10:24 dev-83 systemd[1]: jenkins.service: main process exited, code=exited, status=1/FAILURE
10월 11 15:10:24 dev-83 systemd[1]: Failed to start Jenkins Continuous Integration Server.
```

## resolve

java version이 낮아서 그런 것으로 java 17로 올려서 해결했습니다.