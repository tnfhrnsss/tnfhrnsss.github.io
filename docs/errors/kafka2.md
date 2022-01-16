---
layout: default
title: org.apache.kafka.common.errors.SerializationException
parent: Errors
has_children: false
nav_exclude: true
---

# org.apache.kafka.common.errors.SerializationException: Error deserializing key/value for partition

[에러]

15:04:41.660 er#1-0-C-1 ERROR .k.l.LoggingErrorHandler: 37 handle Error while processing: null[org.apache.kafka.common.errors.SerializationException](https://www.blogger.com/blog/post/edit/2689228726924373128/1302139096699183138#): Error deserializing key/value for partition event.message-0 at offset 62. If needed, please seek past the record to continue consumption.Caused by: org.springframework.messaging.converter.MessageConversionException: failed to resolve class name. Class not found [AAA]; nested exception is java.lang.ClassNotFoundException: AAA at org.springframework.kafka.support.converter.DefaultJackson2JavaTypeMapper.getClassIdType(DefaultJackson2JavaTypeMapper.java:138) at org.springframework.kafka.support.converter.DefaultJackson2JavaTypeMapper.toJavaType(DefaultJackson2JavaTypeMapper.java:99) at org.springframework.kafka.support.serializer.JsonDeserializer.deserialize(JsonDeserializer.java:342) at org.apache.kafka.clients.consumer.internals.Fetcher.parseRecord(Fetcher.java:1041) at org.apache.kafka.clients.consumer.internals.Fetcher.access$3300(Fetcher.java:110) at org.apache.kafka.clients.consumer.internals.Fetcher$PartitionRecords.fetchRecords(Fetcher.java:1223) at org.apache.kafka.clients.consumer.internals.Fetcher$PartitionRecords.access$1400(Fetcher.java:1072) at org.apache.kafka.clients.consumer.internals.Fetcher.fetchRecords(Fetcher.java:562) at org.apache.kafka.clients.consumer.internals.Fetcher.fetchedRecords(Fetcher.java:523) at org.apache.kafka.clients.consumer.KafkaConsumer.pollForFetches(KafkaConsumer.java:1230) at org.apache.kafka.clients.consumer.KafkaConsumer.poll(KafkaConsumer.java:1187) at org.apache.kafka.clients.consumer.KafkaConsumer.poll(KafkaConsumer.java:1115) at brave.kafka.clients.TracingConsumer.poll(TracingConsumer.java:78) at brave.kafka.clients.TracingConsumer.poll(TracingConsumer.java:72) at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:743) at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:700) at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515) at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264) at java.base/java.lang.Thread.run(Thread.java:835)

[원인]

kafka consumer가 producer쪽이랑 header가 달라서, 즉 패키지 path가 달라서 그런거임.해결) kafkalistener에 property를 추가하거나 consumer config에 props.put(JsonDeserializer.USE_TYPE_INFO_HEADERS, false);를 추가한다.@KafkaListener(topics = BBB,groupId = BBB + "-push-subscriber",properties = {"AAA","spring.json.use.type.headers:false"})

[참고]

[http://blog.naver.com/PostView.nhn?blogId=simpolor&logNo=221757494878&parentCategoryNo=&categoryNo=218&viewDate=&isShowPopularPosts=false&from=postViewhttps://stackoverflow.com/questions/55477941/spring-boot-kafka-class-deserialization-not-in-the-trusted-package](https://www.blogger.com/blog/post/edit/2689228726924373128/1302139096699183138#)