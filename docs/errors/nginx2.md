---
layout: post
title: CircularRedirectException
date: 2022-06-16 21:05:00
last_modified_at : 2022-06-16 21:05:00
parent: Errors
has_children: false
nav_exclude: true
tags: [nginx]
---

### error log

```
17:18:17.139 r-CREMA-11 DEBUG .j.JpaTransactionManager:620 AfterCompletion Closing JPA EntityManager [SessionImpl(898129057<open>)] after transaction
17:18:17.173 -gateway-9 DEBUG f.s.Slf4jLogger         : 72 log             [ThirdPartyGatewayFileClient#download] ---> GET http://aaa/attach/download HTTP/1.1
17:18:17.173 -gateway-9 DEBUG f.s.Slf4jLogger         : 72 log             [ThirdPartyGatewayFileClient#download] XAttachUrl: http://bbb
17:18:17.174 -gateway-9 DEBUG f.s.Slf4jLogger         : 72 log             [ThirdPartyGatewayFileClient#download] ---> END HTTP (0-byte body)
17:18:17.185 -gateway-9 DEBUG f.s.Slf4jLogger         : 72 log             [ThirdPartyGatewayFileClient#download] <--- ERROR ClientProtocolException: null (10ms)
17:18:17.186 -gateway-9 DEBUG f.s.Slf4jLogger         : 72 log             [ThirdPartyGatewayFileClient#download] org.apache.http.client.ClientProtocolException
at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:187)
at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:83)
at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:108)
at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:56)
at feign.httpclient.ApacheHttpClient.execute(ApacheHttpClient.java:83)
at org.springframework.cloud.sleuth.instrument.web.client.feign.TracingFeignClient.execute(TracingFeignClient.java:81)
at org.springframework.cloud.sleuth.instrument.web.client.feign.LazyTracingFeignClient.execute(LazyTracingFeignClient.java:60)
at org.springframework.cloud.openfeign.ribbon.RetryableFeignLoadBalancer.lambda$execute$0(RetryableFeignLoadBalancer.java:109)
at org.springframework.retry.support.RetryTemplate.doExecute(RetryTemplate.java:287)
at org.springframework.retry.support.RetryTemplate.execute(RetryTemplate.java:180)
at org.springframework.cloud.openfeign.ribbon.RetryableFeignLoadBalancer.execute(RetryableFeignLoadBalancer.java:92)
at org.springframework.cloud.openfeign.ribbon.RetryableFeignLoadBalancer.execute(RetryableFeignLoadBalancer.java:52)
at com.netflix.client.AbstractLoadBalancerAwareClient$1.call(AbstractLoadBalancerAwareClient.java:104)
at com.netflix.loadbalancer.reactive.LoadBalancerCommand$3$1.call(LoadBalancerCommand.java:303)
at com.netflix.loadbalancer.reactive.LoadBalancerCommand$3$1.call(LoadBalancerCommand.java:287)
at rx.internal.util.ScalarSynchronousObservable$3.call(ScalarSynchronousObservable.java:231)
at rx.internal.util.ScalarSynchronousObservable$3.call(ScalarSynchronousObservable.java:228)
at rx.Observable.unsafeSubscribe(Observable.java:10327)
at rx.internal.operators.OnSubscribeConcatMap$ConcatMapSubscriber.drain(OnSubscribeConcatMap.java:286)
at rx.internal.operators.OnSubscribeConcatMap$ConcatMapSubscriber.onNext(OnSubscribeConcatMap.java:144)
at com.netflix.loadbalancer.reactive.LoadBalancerCommand$1.call(LoadBalancerCommand.java:185)
at com.netflix.loadbalancer.reactive.LoadBalancerCommand$1.call(LoadBalancerCommand.java:180)
at rx.Observable.unsafeSubscribe(Observable.java:10327)
at rx.internal.operators.OnSubscribeConcatMap.call(OnSubscribeConcatMap.java:94)
at rx.internal.operators.OnSubscribeConcatMap.call(OnSubscribeConcatMap.java:42)
at rx.Observable.unsafeSubscribe(Observable.java:10327)
at rx.internal.operators.OperatorRetryWithPredicate$SourceSubscriber$1.call(OperatorRetryWithPredicate.java:127)
at rx.internal.schedulers.TrampolineScheduler$InnerCurrentThreadScheduler.enqueue(TrampolineScheduler.java:73)
at rx.internal.schedulers.TrampolineScheduler$InnerCurrentThreadScheduler.schedule(TrampolineScheduler.java:52)
at rx.internal.operators.OperatorRetryWithPredicate$SourceSubscriber.onNext(OperatorRetryWithPredicate.java:79)
at rx.internal.operators.OperatorRetryWithPredicate$SourceSubscriber.onNext(OperatorRetryWithPredicate.java:45)
at rx.internal.util.ScalarSynchronousObservable$WeakSingleProducer.request(ScalarSynchronousObservable.java:276)
at rx.Subscriber.setProducer(Subscriber.java:209)
at rx.internal.util.ScalarSynchronousObservable$JustOnSubscribe.call(ScalarSynchronousObservable.java:138)
at rx.internal.util.ScalarSynchronousObservable$JustOnSubscribe.call(ScalarSynchronousObservable.java:129)
at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
at rx.Observable.subscribe(Observable.java:10423)
at rx.Observable.subscribe(Observable.java:10390)
at rx.observables.BlockingObservable.blockForSingle(BlockingObservable.java:443)
at rx.observables.BlockingObservable.single(BlockingObservable.java:340)
at com.netflix.client.AbstractLoadBalancerAwareClient.executeWithLoadBalancer(AbstractLoadBalancerAwareClient.java:112)
at org.springframework.cloud.openfeign.ribbon.LoadBalancerFeignClient.execute(LoadBalancerFeignClient.java:84)
at org.springframework.cloud.sleuth.instrument.web.client.feign.TraceLoadBalancerFeignClient.execute(TraceLoadBalancerFeignClient.java:78)
at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:119)
at feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:89)
at feign.hystrix.HystrixInvocationHandler$1.run(HystrixInvocationHandler.java:109)
at com.netflix.hystrix.HystrixCommand$2.call(HystrixCommand.java:302)
at com.netflix.hystrix.HystrixCommand$2.call(HystrixCommand.java:298)
at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:46)
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
at org.springframework.cloud.sleuth.instrument.async.TraceCallable.call(TraceCallable.java:71)
at com.netflix.hystrix.strategy.concurrency.HystrixContexSchedulerAction.call(HystrixContexSchedulerAction.java:69)
at rx.internal.schedulers.ScheduledAction.run(ScheduledAction.java:55)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
at java.util.concurrent.FutureTask.run$$$capture(FutureTask.java:266)
at java.util.concurrent.FutureTask.run(FutureTask.java)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at java.lang.Thread.run(Thread.java:748)
Caused by: org.apache.http.client.CircularRedirectException: Circular redirect to 'http://127.0.0.1:8051/attach/download'
at org.apache.http.impl.client.DefaultRedirectStrategy.getLocationURI(DefaultRedirectStrategy.java:193)
at org.apache.http.impl.client.DefaultRedirectStrategy.getRedirect(DefaultRedirectStrategy.java:223)
at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java:126)
at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:185)
... 81 more
```

### cause

원인은 허무하게도 nginx가 동일 포트로 여러개 떠있었다. 
rewrite 문법이 잘못된 것이 없다면, 프로세스 확인해보장~!  