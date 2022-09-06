---
layout: post
title: cannot be opened because it does not exist
date: 2022-03-27 13:28:45
last_modified_at : 2022-09-06 22:13:00
parent: Errors
has_children: false
nav_exclude: true
tags: [intellij]
---


### problems

인텔리제이에서 spring boot run하면 아래와 같은 오류발생

```
09:06:19.237 main       ERROR o.s.b.SpringApplication :834 reportFailure   Application run failed
org.springframework.beans.factory.BeanDefinitionStoreException: 
Failed to parse configuration class [Application]; 
nested exception is java.io.FileNotFoundException: class path resource 
[ExpireTokenAdapter.class] cannot be opened because it does not exist

Process finished with exit code 1
```

![intellij1.png](../img/intellij1.png)


### try 1
절케해도 안되서

전체 maven reload했는데

java.lang.OutOfMemoryError: GC overhead limit exceeded 이렇게 오류발생

### try 2
메모리 옵션 -xms1024m -xmx2048m으로 늘리고 다시 시도

그래도 에러나는 모듈있어서

에러나는 모듈은 폴더자체 rebuild해주고 다시 재시작하니까 된다

그래도 다음날 계속 소스를 pull받다보니 이미 존재하는 class파일인데 cannot find오류가 나서

springboot 시작전에 before build하는 옵션을 제거하고

#### solution

수동 build하는 것으로 바꾸니 잘된다.

에러도 없고 오히려 이게 빠른것 같다.

수동 build는 인텔리제이 [Build > Build Project] 실행