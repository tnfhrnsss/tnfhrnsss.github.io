---
layout: post
title: com.netflix.client.ClientException
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
tags: [hystrix, msa, feign]
---

### com.netflix.client.ClientException: Load balancer does not have available server for client: aaa

#### cause
feignclient로 다른 모듈 호출하려고 하는데 못찾을 때

#### error log

```
11:46:24.265 010-exec-3 ERROR e.r.BaseExceptionHandler: 57 handleThrowable [status] 500 INTERNAL_SERVER_ERROR, [code]: 500100, [message] Hystrix command fails and does not a fallback., [detail message] SchemaTranslateClient#translate(StandardMessageCdo) failed and no fallback available.

com.netflix.hystrix.exception.HystrixRuntimeException: SchemaTranslateClient#translate(StandardMessageCdo) failed and no fallback available.

at com.netflix.hystrix.AbstractCommand$22.call(AbstractCommand.java:822)
at com.netflix.hystrix.AbstractCommand$22.call(AbstractCommand.java:807)
at rx.internal.operators.OperatorOnErrorResumeNextViaFunction$4.onError(OperatorOnErrorResumeNextViaFunction.java:140)
at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87)
at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87)
at com.netflix.hystrix.AbstractCommand$DeprecatedOnFallbackHookApplication$1.onError(AbstractCommand.java:1472)
at com.netflix.hystrix.AbstractCommand$FallbackHookApplication$1.onError(AbstractCommand.java:1397)
at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87)
at rx.observers.Subscribers$5.onError(Subscribers.java:230)
at rx.internal.operators.OnSubscribeThrow.call(OnSubscribeThrow.java:44)
at rx.internal.operators.OnSubscribeThrow.call(OnSubscribeThrow.java:28)
at rx.Observable.unsafeSubscribe(Observable.java:10327)
at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:51)
...
at rx.internal.operators.OperatorOnErrorResumeNextViaFunction$4.onError(OperatorOnErrorResumeNextViaFunction.java:142)
at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87)
at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87)
at com.netflix.hystrix.AbstractCommand$HystrixObservableTimeoutOperator$3.onError(AbstractCommand.java:1194)
at rx.internal.operators.OperatorSubscribeOn$SubscribeOnSubscriber.onError(OperatorSubscribeOn.java:80)
at rx.observers.Subscribers$5.onError(Subscribers.java:230)


Caused by: java.lang.RuntimeException: com.netflix.client.ClientException: Load balancer does not have available server for client: aaa

at org.springframework.cloud.openfeign.ribbon.LoadBalancerFeignClient.execute(LoadBalancerFeignClient.java:90)
at org.springframework.cloud.sleuth.instrument.web.client.feign.TraceLoadBalancerFeignClient.execute(TraceLoadBalancerFeignClient.java:71)
at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:108)
at feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:78)
at feign.hystrix.HystrixInvocationHandler$1.run(HystrixInvocationHandler.java:109)
at com.netflix.hystrix.HystrixCommand$2.call(HystrixCommand.java:302)
at com.netflix.hystrix.HystrixCommand$2.call(HystrixCommand.java:298)
at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:46)

... 28 common frames omitted
```

#### solution
springboot로 올릴때 해당 모듈이 pom에 있는지랑. springboot application에 feignclient어노테이션이 있는지를 확인한다.