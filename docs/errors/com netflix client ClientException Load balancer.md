---
layout: default
title: com.netflix.client.ClientException
parent: Errors
has_children: false
nav_exclude: true
---

# com.netflix.client.ClientException: Load balancer does not have available server for client: aaa

현상) feignclient로 다른 모듈 호출하려고 하는데 못찾을 때

오류)

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

at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:35)

at rx.Observable.unsafeSubscribe(Observable.java:10327)

at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)

at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)

at rx.Observable.unsafeSubscribe(Observable.java:10327)

at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)

at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)

at rx.Observable.unsafeSubscribe(Observable.java:10327)

at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)

at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)

at rx.Observable.unsafeSubscribe(Observable.java:10327)

at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)

at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)

at rx.Observable.unsafeSubscribe(Observable.java:10327)

at rx.internal.operators.OperatorOnErrorResumeNextViaFunction$4.onError(OperatorOnErrorResumeNextViaFunction.java:142)

at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87)

at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87)

at com.netflix.hystrix.AbstractCommand$HystrixObservableTimeoutOperator$3.onError(AbstractCommand.java:1194)

at rx.internal.operators.OperatorSubscribeOn$SubscribeOnSubscriber.onError(OperatorSubscribeOn.java:80)

at rx.observers.Subscribers$5.onError(Subscribers.java:230)

at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87)

at rx.observers.Subscribers$5.onError(Subscribers.java:230)

at com.netflix.hystrix.AbstractCommand$DeprecatedOnRunHookApplication$1.onError(AbstractCommand.java:1431)

at com.netflix.hystrix.AbstractCommand$ExecutionHookApplication$1.onError(AbstractCommand.java:1362)

at rx.observers.Subscribers$5.onError(Subscribers.java:230)

at rx.observers.Subscribers$5.onError(Subscribers.java:230)

at rx.internal.operators.OnSubscribeThrow.call(OnSubscribeThrow.java:44)

at rx.internal.operators.OnSubscribeThrow.call(OnSubscribeThrow.java:28)

at rx.Observable.unsafeSubscribe(Observable.java:10327)

at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:51)

at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:35)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)

at rx.Observable.unsafeSubscribe(Observable.java:10327)

at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:51)

at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:35)

at rx.Observable.unsafeSubscribe(Observable.java:10327)

at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)

at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)

at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)

at rx.Observable.unsafeSubscribe(Observable.java:10327)

at rx.internal.operators.OperatorSubscribeOn$SubscribeOnSubscriber.call(OperatorSubscribeOn.java:100)

at com.netflix.hystrix.strategy.concurrency.HystrixContexSchedulerAction$1.call(HystrixContexSchedulerAction.java:56)

at com.netflix.hystrix.strategy.concurrency.HystrixContexSchedulerAction$1.call(HystrixContexSchedulerAction.java:47)

at org.springframework.cloud.sleuth.instrument.async.TraceCallable.call(TraceCallable.java:70)

at com.netflix.hystrix.strategy.concurrency.HystrixContexSchedulerAction.call(HystrixContexSchedulerAction.java:69)

at rx.internal.schedulers.ScheduledAction.run(ScheduledAction.java:55)

at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)

at java.base/java.util.concurrent.FutureTask.run$$$capture(FutureTask.java:264)

at java.base/java.util.concurrent.FutureTask.run(FutureTask.java)

at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)

at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)

at java.base/java.lang.Thread.run(Thread.java:835)

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

해결) springboot로 올릴때 해당 모듈이 pom에 있는지랑. springboot application에 feignclient어노테이션이 있는지를 확인한다.