---
layout: post
title: docker hazelcast management-center start error
date: 2022-07-08 22:51:00
last_modified_at : 2022-07-08 22:51:00
parent: Errors
has_children: false
nav_exclude: true
tags: [docker, hazelcast]
description: docker hazelcast management-center start error
keywords: docker
---

### problem
docker에 한번 올리고

다시 재부팅 후 올리면

![hazel_0.png](../img/hazel_0.png)


### error

```
management-center    | ERROR: Could not lock home directory. 
Make sure that Management Center web application is stopped (offline) 
before starting this command. 
If you are sure the application is stopped, 
it means that lock file was not deleted properly. 
Please delete 'mc.lock' file in the home directory manually before using the command.
```

### solved

홈 디렉토리에 있는 mc.lock을 수동으로 삭제하라고 하라는 에러 메시지라서 삭제했다.

management 홈 디렉토리 위치는 INSPECT 화면에서 MC_HOME을 확인한다

![hazel_1.png](../img/hazel_1.png)

서버가 제대로 구동안된 상태이기 때문에, docker의 cli로 못들어가서

도커에 올라가 있는 다른 서비스를 통해서 삭제했던 것 같다.