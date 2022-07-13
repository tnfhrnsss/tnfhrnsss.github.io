---
layout: post
title: kafka SerializationException
date: 2021-09-09 13:07:45
last_modified_at : 2022-07-13 20:34:00
parent: Errors
has_children: false
nav_exclude: true
tags: [kafka, kafdrop]
---

## org.apache.kafka.common.errors.SerializationException: Error deserializing key/value for partition

### error log

```
15:04:41.660 er#1-0-C-1 ERROR .k.l.LoggingErrorHandler: 37 handle Error while processing: 
null[org.apache.kafka.common.errors.SerializationException](https://www.blogger.com/blog/post/edit/2689228726924373128/1302139096699183138#): 

Error deserializing key/value for partition event.message-0 at offset 62. 
If needed, please seek past the record to continue consumption.

Caused by: org.springframework.messaging.converter.MessageConversionException: failed to resolve class name. 

Class not found [AAA]; nested exception is java.lang.ClassNotFoundException: AAA at org.springframework.kafka.support.converter.DefaultJackson2JavaTypeMapper.getClassIdType(DefaultJackson2JavaTypeMapper.java:138) 

at org.springframework.kafka.support.converter.DefaultJackson2JavaTypeMapper.toJavaType(DefaultJackson2JavaTypeMapper.java:99) 
at org.springframework.kafka.support.serializer.JsonDeserializer.deserialize(JsonDeserializer.java:342) 
at org.apache.kafka.clients.consumer.internals.Fetcher.parseRecord(Fetcher.java:1041) 
at org.apache.kafka.clients.consumer.internals.Fetcher.access$3300(Fetcher.java:110) 
at org.apache.kafka.clients.consumer.internals.Fetcher$PartitionRecords.fetchRecords(Fetcher.java:1223) 
at org.apache.kafka.clients.consumer.internals.Fetcher$PartitionRecords.access$1400(Fetcher.java:1072) 
at org.apache.kafka.clients.consumer.internals.Fetcher.fetchRecords(Fetcher.java:562) 
at org.apache.kafka.clients.consumer.internals.Fetcher.fetchedRecords(Fetcher.java:523) 
at org.apache.kafka.clients.consumer.KafkaConsumer.pollForFetches(KafkaConsumer.java:1230) 
at org.apache.kafka.clients.consumer.KafkaConsumer.poll(KafkaConsumer.java:1187) 
at org.apache.kafka.clients.consumer.KafkaConsumer.poll(KafkaConsumer.java:1115) 
at brave.kafka.clients.TracingConsumer.poll(TracingConsumer.java:78) 
at brave.kafka.clients.TracingConsumer.poll(TracingConsumer.java:72) 
at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:743) 
at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:700) 
at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515) 
at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264) 
at java.base/java.lang.Thread.run(Thread.java:835)
```

### cause

kafka consumer가 producer쪽이랑 header가 달라서, 즉 패키지 path가 달라서 그런거임.

### solved1
kafkalistener에 property를 추가하거나 
```
@KafkaListener(    
    topics = KafkaTenant.PREFIX + BusinessTopics.EVENT_MESSAGE,    
    groupId = BusinessTopics.EVENT_MESSAGE + "-crema-business-message-subscriber",    
    properties = {        
        "spring.json.value.default.type:spectra.attic.talk.crema.business.message.send.messaging.event.MessageCreated",        
        "spring.json.use.type.headers:false"   
        }
    )

```

### solved2
consumer config에 
```
props.put(JsonDeserializer.USE_TYPE_INFO_HEADERS, false);
```
를 추가한다.

### solved3
yml에 class를 맵핑시킨다. 단점은 사용하지 않는 이벤트의 헤더까지 맵핑되어야한다.

```
spring:
  kafka:
    consumer:
      properties:
        spring.json.type.mapping: spectra.attic.talk.mocha.btalk.btalk.messaging.event.BtalkCreated:spectra.attic.talk.crema.business.btalk.message.send.messaging.event.TicketCreated,
spectra.attic.talk.mocha.btalk.btalk.messaging.event.BtalkUpdated:spectra.attic.talk.crema.business.btalk.message.send.messaging.event.TicketUpdated

```

단, 해당 이벤트의 모든 헤더를 정의하지 않으면 MessageConversionException 오류 발생
```

Caused by: org.apache.kafka.common.errors.SerializationException: Error deserializing key/value for partition event.mocha.btalk-0 at offset 28. If needed, please seek past the record to continue consumption.
Caused by: org.springframework.messaging.converter.MessageConversionException: failed to resolve class name. Class not found [spectra.attic.talk.mocha.btalk.btalk.messaging.event.BtalkUpdated]; nested exception is java.lang.ClassNotFoundException: spectra.attic.talk.mocha.btalk.btalk.messaging.event.BtalkUpdated
	at org.springframework.kafka.support.converter.DefaultJackson2JavaTypeMapper.getClassIdType(DefaultJackson2JavaTypeMapper.java:139)
	at org.springframework.kafka.support.converter.DefaultJackson2JavaTypeMapper.toJavaType(DefaultJackson2JavaTypeMapper.java:100)
	at org.springframework.kafka.support.serializer.JsonDeserializer.deserialize(JsonDeserializer.java:472)
	at org.apache.kafka.clients.consumer.internals.Fetcher.parseRecord(Fetcher.java:1324)
	at org.apache.kafka.clients.consumer.internals.Fetcher.access$3400(Fetcher.java:129)
	at org.apache.kafka.clients.consumer.internals.Fetcher$CompletedFetch.fetchRecords(Fetcher.java:1555)
	at org.apache.kafka.clients.consumer.internals.Fetcher$CompletedFetch.access$1700(Fetcher.java:1391)
	at org.apache.kafka.clients.consumer.internals.Fetcher.fetchRecords(Fetcher.java:683)
	at org.apache.kafka.clients.consumer.internals.Fetcher.fetchedRecords(Fetcher.java:634)
	at org.apache.kafka.clients.consumer.KafkaConsumer.pollForFetches(KafkaConsumer.java:1289)
	at org.apache.kafka.clients.consumer.KafkaConsumer.poll(KafkaConsumer.java:1243)
	at org.apache.kafka.clients.consumer.KafkaConsumer.poll(KafkaConsumer.java:1173)
	at sun.reflect.GeneratedMethodAccessor344.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:344)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:205)
	at com.sun.proxy.$Proxy340.poll(Unknown Source)
	at brave.kafka.clients.TracingConsumer.poll(TracingConsumer.java:89)
	at brave.kafka.clients.TracingConsumer.poll(TracingConsumer.java:83)
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doPoll(KafkaMessageListenerContainer.java:1246)
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:1146)
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:1059)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run$$$capture(FutureTask.java:266)
	at java.util.concurrent.FutureTask.run(FutureTask.java)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.lang.ClassNotFoundException: 
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:348)
	at org.springframework.util.ClassUtils.forName(ClassUtils.java:284)
	at org.springframework.kafka.support.converter.DefaultJackson2JavaTypeMapper.getClassIdType(DefaultJackson2JavaTypeMapper.java:135)
	... 26 common frames omitted
```

#### reference

- [org.apache.kafka.common.errors.SerializationException: Error deserializing key/value for partition t](http://blog.naver.com/PostView.nhn?blogId=simpolor&logNo=221757494878&parentCategoryNo=&categoryNo=218&viewDate=&isShowPopularPosts=false&from=postView)
- [Spring boot Kafka class deserialization - not in the trusted package](https://stackoverflow.com/questions/55477941/spring-boot-kafka-class-deserialization-not-in-the-trusted-package)