---
layout: post
title: RejectedExecutionException
date: 2022-04-06 20:13:00
last_modified_at : 2022-04-06 20:13:00
parent: Errors
has_children: false
nav_exclude: true
tags: [java, thread]
---


### test scenario

- 1대를 대상으로 크레마 성능시나리오 돌림.
- Xms256m -Xmx256m로 springboot 서비스 올림
- 인입되는 동시 메시지 수 20개
- 크레마 thread는 pool 100개, max 100개, queue 10개로 설정

### error

```
Exception in KakaoMessageController.message() 
with cause = 'java.util.concurrent.RejectedExecutionException: 
Task java.util.concurrent.FutureTask@249a9d22

rejected from java.util.concurrent.ThreadPoolExecutor@3b96a13d
[Running, pool size = 100, active threads = 1, queued tasks = 10, 
completed tasks = 9975]'

and exception = 'Executor [java.util.concurrent.ThreadPoolExecutor@3b96a13d
[Running, pool size = 100, active threads = 8, queued tasks = 5, 
completed tasks = 9975]]

did not accept task: org.springframework.cloud.sleuth.instrument.async.
TraceCallable@aaea32a'org.springframework.core.task.TaskRejectedException:

Executor [java.util.concurrent.ThreadPoolExecutor@3b96a13d
[Running, pool size = 100, active threads = 8, queued tasks = 5, 
completed tasks = 9975]]
```

### problem

서버상태) cpu는 정상, 다른서비스도 정상

조치)

- jvm메모리도 작은 상황에 큐가 max가 되어서 rejectException이 발생한 것으로 보임
    - 방법1. rejectexceptionhandler에 policy를 준다 ([https://stackoverflow.com/questions/49290054/taskrejectedexception-in-threadpooltaskexecutor](https://stackoverflow.com/questions/49290054/taskrejectedexception-in-threadpooltaskexecutor))
        - callreturnpolicy를 쓰면 호출한 쓰레드로 동작을 하게하고, 그 쓰레드가 종료될 때까지 다른 호출은 지연된다는 문제
        - queue가 꽉찬 상황에서 처리한다해도, 문제가 될 것으로 보임. queue가 왜 꽉 차서 안사라지는지 확인이 필요
            - 20개의 쓰레드가 동시에 호출된 문제.
- queue사이즈는 적당한가??
    - [https://stackoverflow.com/a/43874563/14257397](https://stackoverflow.com/a/43874563/14257397)
    - acceptable time range / time to complete a task.

### conclusion

- queue size를 50으로 올린 후, jmx를 1024로 상향
- 반드시 maxpool만큼 채워서 쓰는게 아니라고 함. 즉, 실제로 corePoolSize가 스레드 개수보다 많다고 해서 maximumPoolSize 개수 까지 바로 생성하지 않는다. 그 전에 큐(workQueue)에 담고 대기한다. (구현체마다 다르지만, 일반적으로)
- 그리고 나서 workQueue에도 담을 공간이 부족하다면 그때 maximumPoolSize 까지 스레드를 늘려 작업을 한다. 그 후 keepAliveTime에 도달하면 다시 corePoolSize 로 유지 된다.
- 만약 queue가 다 찼으면 maxpoolsize만큼 생성한다.
- 그러다 모든 쓰레드가 사용중이고 queue도 full이면 거부된다.