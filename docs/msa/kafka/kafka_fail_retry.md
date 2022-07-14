---
layout: post
title: kafka fail retry & backoff
date: 2022-07-14 22:58:00
last_modified_at : 2022-07-14 22:58:00
parent: kafka
grand_parent: Msa
nav_exclude: true
tags: [kafka, retry]
---

## cause

consumer 클래스에서 예외상황이 발생할 경우, kafka는 재시도를 한다.

batchkafka는 SeekToCurrentBatchErrorHandler를 통해서 무한반복

kafka는 SeekToCurrentErrorHandler를 통해서 retry를 20번 반복

밑에 에러는 key exception이 발생해서 반복적으로 에러가 발생했던 상황

## error

```prolog
08:16:36.414 010-exec-3 DEBUG .ControllerLoggingAspect: 71 logAround       Exit: spectra.attic.talk.crema.thirdparty.kakao.message.receive.controller.KakaoMessageController.reference() with result null

08:16:36.560 er#8-0-C-1 ERROR o.s.c.l.LogAccessor     :149 error           Error handler threw an exception

org.springframework.kafka.KafkaException: Seek to current after exception; nested exception is org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(java.util.List<spectra.attic.talk.crema.thirdparty.kakao.message.domain.receive.KakaoMessage>)' threw exception; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at org.springframework.kafka.listener.**SeekToCurrentBatchErrorHandler.handle(SeekToCurrentBatchErrorHandler.java:72)**

at org.springframework.kafka.listener.RecoveringBatchErrorHandler.handle(RecoveringBatchErrorHandler.java:124)

at org.springframework.kafka.listener.ContainerAwareBatchErrorHandler.handle(ContainerAwareBatchErrorHandler.java:56)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchErrorHandler(KafkaMessageListenerContainer.java:1776)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchListener(KafkaMessageListenerContainer.java:1648)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchListener(KafkaMessageListenerContainer.java:1519)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeListener(KafkaMessageListenerContainer.java:1502)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:1164)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:1059)

at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)

at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)

at java.base/java.lang.Thread.run(Thread.java:835)

Caused by: org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(java.util.List<spectra.attic.talk.crema.thirdparty.kakao.message.domain.receive.KakaoMessage>)' threw exception; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.decorateException(KafkaMessageListenerContainer.java:2084)

... 9 common frames omitted

Caused by: spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at spectra.attic.talk.crema.thirdparty.message.receive.service.flow.ThirdPartyMessageFlowService.opened(ThirdPartyMessageFlowService.java:33)

at spectra.attic.talk.crema.thirdparty.message.receive.adapter.local.ThirdPartyMessageLocalAdapter.opened(ThirdPartyMessageLocalAdapter.java:21)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService.opened(KakaoMessageReceiveFlowService.java:33)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService$$FastClassBySpringCGLIB$$bee8a510.invoke(<generated>)

at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:771)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor.invoke(MethodBeforeAdviceInterceptor.java:56)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:175)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:95)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:691)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService$$EnhancerBySpringCGLIB$$3892125f.opened(<generated>)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.opened(KakaoMessageEventSubscriber.java:78)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.lambda$received$0(KakaoMessageEventSubscriber.java:57)

at java.base/java.util.Optional.orElseGet(Optional.java:369)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.received(KakaoMessageEventSubscriber.java:57)

at java.base/java.util.ArrayList.forEach(ArrayList.java:1540)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(KakaoMessageEventSubscriber.java:50)

at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)

at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)

at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)

at java.base/java.lang.reflect.Method.invoke(Method.java:567)

at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:171)

at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.invoke(InvocableHandlerMethod.java:120)

at org.springframework.kafka.listener.adapter.DelegatingInvocableHandler.invoke(DelegatingInvocableHandler.java:221)

at org.springframework.kafka.listener.adapter.HandlerAdapter.invoke(HandlerAdapter.java:55)

at org.springframework.kafka.listener.adapter.MessagingMessageListenerAdapter.invokeHandler(MessagingMessageListenerAdapter.java:321)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.invoke(BatchMessagingMessageListenerAdapter.java:170)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.onMessage(BatchMessagingMessageListenerAdapter.java:162)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.onMessage(BatchMessagingMessageListenerAdapter.java:58)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchOnMessage(KafkaMessageListenerContainer.java:1753)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchOnMessageWithRecordsOrList(KafkaMessageListenerContainer.java:1744)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchOnMessage(KafkaMessageListenerContainer.java:1700)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchListener(KafkaMessageListenerContainer.java:1629)

... 7 common frames omitted

08:16:37.070 er#8-0-C-1 ERROR o.s.c.l.LogAccessor     :149 error           Error handler threw an exception

org.springframework.kafka.KafkaException: Seek to current after exception; nested exception is org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(java.util.List<spectra.attic.talk.crema.thirdparty.kakao.message.domain.receive.KakaoMessage>)' threw exception; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at org.springframework.kafka.listener.SeekToCurrentBatchErrorHandler.handle(SeekToCurrentBatchErrorHandler.java:72)

at org.springframework.kafka.listener.RecoveringBatchErrorHandler.handle(RecoveringBatchErrorHandler.java:124)

at org.springframework.kafka.listener.ContainerAwareBatchErrorHandler.handle(ContainerAwareBatchErrorHandler.java:56)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchErrorHandler(KafkaMessageListenerContainer.java:1776)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchListener(KafkaMessageListenerContainer.java:1648)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchListener(KafkaMessageListenerContainer.java:1519)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeListener(KafkaMessageListenerContainer.java:1502)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:1164)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:1059)

at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)

at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)

at java.base/java.lang.Thread.run(Thread.java:835)

Caused by: org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(java.util.List<spectra.attic.talk.crema.thirdparty.kakao.message.domain.receive.KakaoMessage>)' threw exception; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.decorateException(KafkaMessageListenerContainer.java:2084)

... 9 common frames omitted

Caused by: spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at spectra.attic.talk.crema.thirdparty.message.receive.service.flow.ThirdPartyMessageFlowService.opened(ThirdPartyMessageFlowService.java:33)

at spectra.attic.talk.crema.thirdparty.message.receive.adapter.local.ThirdPartyMessageLocalAdapter.opened(ThirdPartyMessageLocalAdapter.java:21)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService.opened(KakaoMessageReceiveFlowService.java:33)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService$$FastClassBySpringCGLIB$$bee8a510.invoke(<generated>)

at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:771)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor.invoke(MethodBeforeAdviceInterceptor.java:56)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:175)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:95)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:691)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService$$EnhancerBySpringCGLIB$$3892125f.opened(<generated>)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.opened(KakaoMessageEventSubscriber.java:78)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.lambda$received$0(KakaoMessageEventSubscriber.java:57)

at java.base/java.util.Optional.orElseGet(Optional.java:369)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.received(KakaoMessageEventSubscriber.java:57)

at java.base/java.util.ArrayList.forEach(ArrayList.java:1540)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(KakaoMessageEventSubscriber.java:50)

at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)

at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)

at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)

at java.base/java.lang.reflect.Method.invoke(Method.java:567)

at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:171)

at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.invoke(InvocableHandlerMethod.java:120)

at org.springframework.kafka.listener.adapter.DelegatingInvocableHandler.invoke(DelegatingInvocableHandler.java:221)

at org.springframework.kafka.listener.adapter.HandlerAdapter.invoke(HandlerAdapter.java:55)

at org.springframework.kafka.listener.adapter.MessagingMessageListenerAdapter.invokeHandler(MessagingMessageListenerAdapter.java:321)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.invoke(BatchMessagingMessageListenerAdapter.java:170)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.onMessage(BatchMessagingMessageListenerAdapter.java:162)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.onMessage(BatchMessagingMessageListenerAdapter.java:58)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchOnMessage(KafkaMessageListenerContainer.java:1753)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchOnMessageWithRecordsOrList(KafkaMessageListenerContainer.java:1744)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchOnMessage(KafkaMessageListenerContainer.java:1700)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchListener(KafkaMessageListenerContainer.java:1629)

... 7 common frames omitted

08:16:37.582 er#8-0-C-1 ERROR o.s.c.l.LogAccessor     :149 error           Error handler threw an exception

org.springframework.kafka.KafkaException: Seek to current after exception; nested exception is org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(java.util.List<spectra.attic.talk.crema.thirdparty.kakao.message.domain.receive.KakaoMessage>)' threw exception; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at org.springframework.kafka.listener.SeekToCurrentBatchErrorHandler.handle(SeekToCurrentBatchErrorHandler.java:72)

at org.springframework.kafka.listener.RecoveringBatchErrorHandler.handle(RecoveringBatchErrorHandler.java:124)

at org.springframework.kafka.listener.ContainerAwareBatchErrorHandler.handle(ContainerAwareBatchErrorHandler.java:56)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchErrorHandler(KafkaMessageListenerContainer.java:1776)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchListener(KafkaMessageListenerContainer.java:1648)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchListener(KafkaMessageListenerContainer.java:1519)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeListener(KafkaMessageListenerContainer.java:1502)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:1164)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:1059)

at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)

at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)

at java.base/java.lang.Thread.run(Thread.java:835)

Caused by: org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(java.util.List<spectra.attic.talk.crema.thirdparty.kakao.message.domain.receive.KakaoMessage>)' threw exception; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.decorateException(KafkaMessageListenerContainer.java:2084)

... 9 common frames omitted

Caused by: spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at spectra.attic.talk.crema.thirdparty.message.receive.service.flow.ThirdPartyMessageFlowService.opened(ThirdPartyMessageFlowService.java:33)

at spectra.attic.talk.crema.thirdparty.message.receive.adapter.local.ThirdPartyMessageLocalAdapter.opened(ThirdPartyMessageLocalAdapter.java:21)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService.opened(KakaoMessageReceiveFlowService.java:33)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService$$FastClassBySpringCGLIB$$bee8a510.invoke(<generated>)

at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:771)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor.invoke(MethodBeforeAdviceInterceptor.java:56)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:175)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:95)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:691)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService$$EnhancerBySpringCGLIB$$3892125f.opened(<generated>)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.opened(KakaoMessageEventSubscriber.java:78)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.lambda$received$0(KakaoMessageEventSubscriber.java:57)

at java.base/java.util.Optional.orElseGet(Optional.java:369)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.received(KakaoMessageEventSubscriber.java:57)

at java.base/java.util.ArrayList.forEach(ArrayList.java:1540)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(KakaoMessageEventSubscriber.java:50)

at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)

at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)

at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)

at java.base/java.lang.reflect.Method.invoke(Method.java:567)

at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:171)

at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.invoke(InvocableHandlerMethod.java:120)

at org.springframework.kafka.listener.adapter.DelegatingInvocableHandler.invoke(DelegatingInvocableHandler.java:221)

at org.springframework.kafka.listener.adapter.HandlerAdapter.invoke(HandlerAdapter.java:55)

at org.springframework.kafka.listener.adapter.MessagingMessageListenerAdapter.invokeHandler(MessagingMessageListenerAdapter.java:321)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.invoke(BatchMessagingMessageListenerAdapter.java:170)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.onMessage(BatchMessagingMessageListenerAdapter.java:162)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.onMessage(BatchMessagingMessageListenerAdapter.java:58)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchOnMessage(KafkaMessageListenerContainer.java:1753)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchOnMessageWithRecordsOrList(KafkaMessageListenerContainer.java:1744)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchOnMessage(KafkaMessageListenerContainer.java:1700)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchListener(KafkaMessageListenerContainer.java:1629)

... 7 common frames omitted

08:16:38.093 er#8-0-C-1 ERROR o.s.c.l.LogAccessor     :149 error           Error handler threw an exception

org.springframework.kafka.KafkaException: Seek to current after exception; nested exception is org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(java.util.List<spectra.attic.talk.crema.thirdparty.kakao.message.domain.receive.KakaoMessage>)' threw exception; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at org.springframework.kafka.listener.SeekToCurrentBatchErrorHandler.handle(SeekToCurrentBatchErrorHandler.java:72)

at org.springframework.kafka.listener.RecoveringBatchErrorHandler.handle(RecoveringBatchErrorHandler.java:124)

at org.springframework.kafka.listener.ContainerAwareBatchErrorHandler.handle(ContainerAwareBatchErrorHandler.java:56)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchErrorHandler(KafkaMessageListenerContainer.java:1776)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchListener(KafkaMessageListenerContainer.java:1648)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchListener(KafkaMessageListenerContainer.java:1519)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeListener(KafkaMessageListenerContainer.java:1502)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:1164)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:1059)

at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)

at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)

at java.base/java.lang.Thread.run(Thread.java:835)

Caused by: org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(java.util.List<spectra.attic.talk.crema.thirdparty.kakao.message.domain.receive.KakaoMessage>)' threw exception; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.decorateException(KafkaMessageListenerContainer.java:2084)

... 9 common frames omitted

Caused by: spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at spectra.attic.talk.crema.thirdparty.message.receive.service.flow.ThirdPartyMessageFlowService.opened(ThirdPartyMessageFlowService.java:33)

at spectra.attic.talk.crema.thirdparty.message.receive.adapter.local.ThirdPartyMessageLocalAdapter.opened(ThirdPartyMessageLocalAdapter.java:21)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService.opened(KakaoMessageReceiveFlowService.java:33)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService$$FastClassBySpringCGLIB$$bee8a510.invoke(<generated>)

at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:771)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor.invoke(MethodBeforeAdviceInterceptor.java:56)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:175)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:95)

at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)

at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)

at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:691)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.service.flow.KakaoMessageReceiveFlowService$$EnhancerBySpringCGLIB$$3892125f.opened(<generated>)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.opened(KakaoMessageEventSubscriber.java:78)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.lambda$received$0(KakaoMessageEventSubscriber.java:57)

at java.base/java.util.Optional.orElseGet(Optional.java:369)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.received(KakaoMessageEventSubscriber.java:57)

at java.base/java.util.ArrayList.forEach(ArrayList.java:1540)

at spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(KakaoMessageEventSubscriber.java:50)

at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)

at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)

at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)

at java.base/java.lang.reflect.Method.invoke(Method.java:567)

at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:171)

at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.invoke(InvocableHandlerMethod.java:120)

at org.springframework.kafka.listener.adapter.DelegatingInvocableHandler.invoke(DelegatingInvocableHandler.java:221)

at org.springframework.kafka.listener.adapter.HandlerAdapter.invoke(HandlerAdapter.java:55)

at org.springframework.kafka.listener.adapter.MessagingMessageListenerAdapter.invokeHandler(MessagingMessageListenerAdapter.java:321)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.invoke(BatchMessagingMessageListenerAdapter.java:170)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.onMessage(BatchMessagingMessageListenerAdapter.java:162)

at org.springframework.kafka.listener.adapter.BatchMessagingMessageListenerAdapter.onMessage(BatchMessagingMessageListenerAdapter.java:58)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchOnMessage(KafkaMessageListenerContainer.java:1753)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchOnMessageWithRecordsOrList(KafkaMessageListenerContainer.java:1744)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchOnMessage(KafkaMessageListenerContainer.java:1700)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchListener(KafkaMessageListenerContainer.java:1629)

... 7 common frames omitted

08:16:38.604 er#8-0-C-1 ERROR o.s.c.l.LogAccessor     :149 error           Error handler threw an exception

org.springframework.kafka.KafkaException: Seek to current after exception; nested exception is org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(java.util.List<spectra.attic.talk.crema.thirdparty.kakao.message.domain.receive.KakaoMessage>)' threw exception; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at org.springframework.kafka.listener.SeekToCurrentBatchErrorHandler.handle(SeekToCurrentBatchErrorHandler.java:72)

at org.springframework.kafka.listener.RecoveringBatchErrorHandler.handle(RecoveringBatchErrorHandler.java:124)

at org.springframework.kafka.listener.ContainerAwareBatchErrorHandler.handle(ContainerAwareBatchErrorHandler.java:56)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchErrorHandler(KafkaMessageListenerContainer.java:1776)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeBatchListener(KafkaMessageListenerContainer.java:1648)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeBatchListener(KafkaMessageListenerContainer.java:1519)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeListener(KafkaMessageListenerContainer.java:1502)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:1164)

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:1059)

at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)

at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)

at java.base/java.lang.Thread.run(Thread.java:835)

Caused by: org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void spectra.attic.talk.crema.thirdparty.kakao.message.receive.subscriber.KakaoMessageEventSubscriber.handle(java.util.List<spectra.attic.talk.crema.thirdparty.kakao.message.domain.receive.KakaoMessage>)' threw exception; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa; nested exception is spectra.attic.talk.crema.conversation.setting.domain.exception.NotAllowedKeyException: Not Allowed Key Exception : e3158759b0449439fe59b2edb32b4de91bce13fa

at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.decorateException(KafkaMessageListenerContainer.java:2084)

... 9 common frames omitted
```


## solve1

```java

@Bean

public ConcurrentKafkaListenerContainerFactory<String, JsonSerializable> kafkaCremaListenerContainerFactory() {

*/* final SeekToCurrentErrorHandler errorHandler = new SeekToCurrentErrorHandler();*

*errorHandler.set(new FixedBackOff(2000, 0L));*

- */*

*// SeekToCurrentErrorHandler eh = new SeekToCurrentErrorHandler((rec, ex) -> System.out.println("I am the dlq"), new FixedBackOff(0L, 0));*

ConcurrentKafkaListenerContainerFactory<String, JsonSerializable> factory = new ConcurrentKafkaListenerContainerFactory<>();

factory.setConsumerFactory(cconsumerFactory());

factory.setConcurrency(concurrency);

*// factory.setErrorHandler(eh);*

*//factory.setErrorHandler(new SeekToCurrentErrorHandler(new FixedBackOff(0L, 0)));*

factory.setErrorHandler(new SeekToCurrentErrorHandler(0));

return factory;

}

@Bean

public ConcurrentKafkaListenerContainerFactory<String, JsonSerializable> batchCremaKafkaListenerContainerFactory() {

*// final SeekToCurrentBatchErrorHandler errorHandler = new SeekToCurrentBatchErrorHandler();*

*//errorHandler.setBackOff(new FixedBackOff(0L, 0));*

ConcurrentKafkaListenerContainerFactory<String, JsonSerializable> factory = new ConcurrentKafkaListenerContainerFactory<>();

factory.setConsumerFactory(cconsumerFactory());

factory.setConcurrency(concurrency);

factory.setBatchListener(true);

*//SeekToCurrentErrorHandler eh = new SeekToCurrentErrorHandler((rec, ex) -> System.out.println("I am the dlq"), new ExponentialBackOff());*

*// factory.setBatchErrorHandler(errorHandler);*

return factory;

}
```

## solve2

에러발생이 서버장애로 인한 것이라면 재시도가 맞겠지만,

데이터 문제로 뱉는 에러라서 재시도가 필요없을 경우는

backoff기능 외에 subscribe하는 consumer클래스에서 try~catch로 잡는 방법이 있다.

## reference

[SeekToCurrentErrorHandler + ExponentialBackOff will log errors after backOff has passed, is that intentional?](https://stackoverflow.com/questions/64937593/seektocurrenterrorhandler-exponentialbackoff-will-log-errors-after-backoff-has)