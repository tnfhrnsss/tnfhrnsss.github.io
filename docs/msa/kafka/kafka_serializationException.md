---
layout: post
title: kafka SerializationException
date: 2021-09-09 13:07:45
last_modified_at : 2022-09-20 20:34:00
parent: Kafka
grand_parent: Msa
nav_exclude: true
tags: [kafka, kafdrop, serialization]
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

kafka consumer가 producer쪽이랑 header가 달라서, 즉 패키지 path가 달라서 그런거임.

### solved1
kafkalistener에 property를 추가하거나 
```
@KafkaListener(    
    topics = KafkaTenant.PREFIX + BusinessTopics.EVENT_MESSAGE,    
    groupId = BusinessTopics.EVENT_MESSAGE + "-crema-business-message-subscriber",    
    properties = {        
        "spring.json.value.default.type:messaging.event.MessageCreated",        
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
        spring.json.type.mapping: messaging.event.BtalkCreated:messaging.event.TicketCreated,
messaging.event.BtalkUpdated:messaging.event.TicketUpdated

```

단, 해당 이벤트의 모든 헤더를 정의하지 않으면 아까와 동일한 SerializationException, MessageConversionException 에러 발생

### solved4
또 다른 방법은 이벤트를 수신하는 클래스의 @KafkaListener에 spring.json.type.mapping을 설정하는 것

```
@KafkaListener(
    topics = {
        "TestTopic"
    },
    groupId = "test-event-subscriber",
    properties = {
        "spring.json.type.mapping:"
            + "messaging.event.BtalkCreated:messaging.event.TicketCreated,"
            + "messaging.event.BtalkUpdated:messaging.event.TicketUpdated"
    }
)
```

- 만약에 이렇게 했는데 class not found exception이 발생한다면 spring.json.type.mapping로 설정한 여러 클래스들을 한 줄로 만든담에 개행을 해보도록 한다.

### reference

- [org.apache.kafka.common.errors.SerializationException: Error deserializing key/value for partition t](http://blog.naver.com/PostView.nhn?blogId=simpolor&logNo=221757494878&parentCategoryNo=&categoryNo=218&viewDate=&isShowPopularPosts=false&from=postView)
- [Spring boot Kafka class deserialization - not in the trusted package](https://stackoverflow.com/questions/55477941/spring-boot-kafka-class-deserialization-not-in-the-trusted-package)