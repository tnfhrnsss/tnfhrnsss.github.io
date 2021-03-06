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

kafka consumer??? producer????????? header??? ?????????, ??? ????????? path??? ????????? ????????????.

### solved1
kafkalistener??? property??? ??????????????? 
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
consumer config??? 
```
props.put(JsonDeserializer.USE_TYPE_INFO_HEADERS, false);
```
??? ????????????.

### solved3
yml??? class??? ???????????????. ????????? ???????????? ?????? ???????????? ???????????? ?????????????????????.

```
spring:
  kafka:
    consumer:
      properties:
        spring.json.type.mapping: spectra.attic.talk.mocha.btalk.btalk.messaging.event.BtalkCreated:spectra.attic.talk.crema.business.btalk.message.send.messaging.event.TicketCreated,
spectra.attic.talk.mocha.btalk.btalk.messaging.event.BtalkUpdated:spectra.attic.talk.crema.business.btalk.message.send.messaging.event.TicketUpdated

```

???, ?????? ???????????? ?????? ????????? ???????????? ????????? ????????? ????????? SerializationException, MessageConversionException ?????? ??????

????????? ??? ????????? ????????? yml??? ??????????????????..


#### reference

- [org.apache.kafka.common.errors.SerializationException: Error deserializing key/value for partition t](http://blog.naver.com/PostView.nhn?blogId=simpolor&logNo=221757494878&parentCategoryNo=&categoryNo=218&viewDate=&isShowPopularPosts=false&from=postView)
- [Spring boot Kafka class deserialization - not in the trusted package](https://stackoverflow.com/questions/55477941/spring-boot-kafka-class-deserialization-not-in-the-trusted-package)