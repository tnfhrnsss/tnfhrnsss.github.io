---
layout: post
title: TimedSupervisorTask task supervisor timed out - TimeoutException
date: 2022-07-16 14:04:00
last_modified_at : 2022-07-16 14:04:00
parent: Errors
has_children: false
nav_exclude: true
tags: [java, springboot]
---

## problem

서비스 올릴 때 발생하는 오류로 오류와 무관하게 서비스는 잘 되고 있다.

## error

```prolog
07:53:02.003 ryClient-0 WARN  .n.d.TimedSupervisorTask: 73 run task supervisor timed out
java.util.concurrent.TimeoutException: null
at java.util.concurrent.FutureTask.get(FutureTask.java:205)
at com.netflix.discovery.TimedSupervisorTask.run(TimedSupervisorTask.java:68)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
at java.util.concurrent.FutureTask.run(FutureTask.java:266)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at java.lang.Thread.run(Thread.java:748)
07:53:17.011 ryClient-0 WARN  .n.d.TimedSupervisorTask: 73 run task supervisor timed out
java.util.concurrent.TimeoutException: null
at java.util.concurrent.FutureTask.get(FutureTask.java:205)
at com.netflix.discovery.TimedSupervisorTask.run(TimedSupervisorTask.java:68)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
at java.util.concurrent.FutureTask.run(FutureTask.java:266)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at java.lang.Thread.run(Thread.java:748)
07:53:25.348 main       DEBUG s.a.c.s.c.c.CipherConfig: 19 cipherCrypto  CipherCrypto:CipherCryptoProperties{encryptAlgorithm='AES/ECB/PKCS5Padding'}
07:53:25.363 main       DEBUG s.a.c.s.c.c.JsonCrypto  : 30 <init>          JsonCrypto[CipherCrypto], CryptoProperties{enabled=true, encryptFields=[message, metadata.address, name, email, phone, extra, answerValues]}
07:53:37.015 ryClient-0 WARN  .n.d.TimedSupervisorTask: 84 run task supervisor rejected the task
java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.FutureTask@354aaa21 rejected from java.util.concurrent.ThreadPoolExecutor@704822ec[Running, pool size = 2, active threads = 2, queued tasks = 0, completed tasks = 0]
at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2063)
at java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:830)
at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1379)
at java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:112)
at com.netflix.discovery.TimedSupervisorTask.run(TimedSupervisorTask.java:66)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
at java.util.concurrent.FutureTask.run(FutureTask.java:266)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at java.lang.Thread.run(Thread.java:748)
```

## try 1 (실패)

application.yml에 아래 속성 추가

```yaml
eureka:
  client:
    fetch-registry: true
    register-with-eureka: true
```

[https://stackoverflow.com/questions/37542939/problems-spring-cloud-config-and-spring-cloud-bus](https://stackoverflow.com/questions/37542939/problems-spring-cloud-config-and-spring-cloud-bus)

## try 2

eureka버그로 버전 업 필요

## try 3

client.registryFetchIntervalSeconds를 30초로 늘려주라고 함

30초로 늘려도 발생해서 60초로 설정함.

60초로 늘려도 발생해서..

## cause

[https://www.cnblogs.com/linus-tan/p/14814045.html](https://www.cnblogs.com/linus-tan/p/14814045.html)

여기서 말하는 원인은

```
프로젝트가 시작되고 초기화되면 두 가지 타이밍 작업이 시작됩니다. 
하나는 유레카에 등록하는 것, 즉 하트비트 스레드이고, 
다른 하나는 유레카에서 서비스 등록 목록을 가져오는 것, 즉 새로 고칠 스레드입니다. 
등록 목록 캐시.위의 두 스레드는 30초마다 실행되며 기본 타임아웃 시간은 30초이므로 
30초 이내에 작업이 완료되지 않으면 Timeout 예외가 발생합니다. 

하트비트를 보내는 작업은 비교적 간단하며 일반적으로 시간이 초과되지 않습니다. 
등록 목록을 얻는 작업은 오랜 시간이 걸립니다.

레지스트리는 모든 서비스 통신의 기반이므로 여러 서버에 배포된 응용 프로그램은 통신하는 데 
시간이 걸릴 수 있으며 시간 초과가 자주 발생합니다.
```