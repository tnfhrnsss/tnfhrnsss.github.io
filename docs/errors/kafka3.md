---
layout: post
title: org.springframework.kafka.KafkaException
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
tags: [kafka]
---

### org.springframework.kafka.KafkaException: Ambiguous methods for payload type

#### error log

```
org.springframework.kafka.KafkaException: Ambiguous methods for payload type:** class AAA: handle and handle

at org.springframework.kafka.listener.adapter.DelegatingInvocableHandler.findHandlerForPayload(DelegatingInvocableHandler.java:233)
at org.springframework.kafka.listener.adapter.DelegatingInvocableHandler.getHandlerForPayload(DelegatingInvocableHandler.java:168)
at org.springframework.kafka.listener.adapter.DelegatingInvocableHandler.getMethodNameFor(DelegatingInvocableHandler.java:279)
at org.springframework.kafka.listener.adapter.HandlerAdapter.getMethodAsString(HandlerAdapter.java:67)
at org.springframework.kafka.listener.adapter.MessagingMessageListenerAdapter.invokeHandler(MessagingMessageListenerAdapter.java:302)
at org.springframework.kafka.listener.adapter.RecordMessagingMessageListenerAdapter.onMessage(RecordMessagingMessageListenerAdapter.java:79)
at org.springframework.kafka.listener.adapter.RecordMessagingMessageListenerAdapter.onMessage(RecordMessagingMessageListenerAdapter.java:50)
at org.springframework.kafka.listener.adapter.RecordMessagingMessageListenerAdapter$$FastClassBySpringCGLIB$$cde8c01d.invoke(<generated>)
at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:749)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
at org.springframework.cloud.sleuth.instrument.messaging.MessageListenerMethodInterceptor.invoke(TraceMessagingAutoConfiguration.java:324)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:688)
at org.springframework.kafka.listener.adapter.RecordMessagingMessageListenerAdapter$$EnhancerBySpringCGLIB$$4dfa1e78.onMessage(<generated>)
at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeOnMessage(KafkaMessageListenerContainer.java:1275)
at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeOnMessage(KafkaMessageListenerContainer.java:1258)
at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeRecordListener(KafkaMessageListenerContainer.java:1219)
at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeWithRecords(KafkaMessageListenerContainer.java:1200)
at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeRecordListener(KafkaMessageListenerContainer.java:1120)
at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeListener(KafkaMessageListenerContainer.java:935)
at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:751)
at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:700)
at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
at java.base/java.lang.Thread.run(Thread.java:835)

Process finished with exit code -1
```


#### cause
내가 handle메소드 인자를 Object타입으로 받은거랑 MessageCreated로 받은걸 두개 생성해서... 어디에 매핑할지 모호하다는 에러임. Object인자로 받는 메소드를 지우자