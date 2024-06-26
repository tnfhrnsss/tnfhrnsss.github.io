---
layout: post
title: kafka fail retry & backoff (how to handle SeekToCurrentBatchError)
date: 2022-07-14 22:58:00
last_modified_at : 2022-07-14 22:58:00
parent: Kafka
grand_parent: Msa
nav_exclude: true
tags: [kafka, retry]
---

## Problem

consumer 클래스에서 예외 상황이 발생할 경우, kafka는 재시도를 한다.

batch kafka는 SeekToCurrentBatchErrorHandler를 통해서 무한반복

kafka는 SeekToCurrentErrorHandler를 통해서 retry를 20번 반복

밑에 에러는 key exception이 발생해서 반복적으로 에러가 발생했던 상황

## Error

```prolog
08:16:36.560 er#8-0-C-1 ERROR o.s.c.l.LogAccessor:149 error Error handler threw an exception
org.springframework.kafka.KafkaException: 
Seek to current after exception; 
nested exception is org.springframework.kafka.listener.ListenerExecutionFailedException: 
Listener method 'public void KakaoMessageEventSubscriber.handle(java.util.List<KakaoMessage>)' 
threw exception; 
```


## Try FixedBackOff Config

- kafka consumer config로 ErrorHandler에 FixedBackOff를 설정하는 하는 것으로 제어할 수 있다.

```java

@Bean
public ConcurrentKafkaListenerContainerFactory<String, JsonSerializable> kafkaCremaListenerContainerFactory() {
// final SeekToCurrentErrorHandler errorHandler = new SeekToCurrentErrorHandler();
// errorHandler.set(new FixedBackOff(2000, 0L));
// SeekToCurrentErrorHandler eh = new SeekToCurrentErrorHandler((rec, ex) -> System.out.println("I am the dlq"), new FixedBackOff(0L, 0));

ConcurrentKafkaListenerContainerFactory<String, JsonSerializable> factory = new ConcurrentKafkaListenerContainerFactory<>();
factory.setConsumerFactory(cconsumerFactory());
factory.setConcurrency(concurrency);

// factory.setErrorHandler(eh);
//factory.setErrorHandler(new SeekToCurrentErrorHandler(new FixedBackOff(0L, 0)));

factory.setErrorHandler(new SeekToCurrentErrorHandler(0));
return factory;

}

@Bean
public ConcurrentKafkaListenerContainerFactory<String, JsonSerializable> batchCremaKafkaListenerContainerFactory() {
// final SeekToCurrentBatchErrorHandler errorHandler = new SeekToCurrentBatchErrorHandler();
//errorHandler.setBackOff(new FixedBackOff(0L, 0));

ConcurrentKafkaListenerContainerFactory<String, JsonSerializable> factory = new ConcurrentKafkaListenerContainerFactory<>();
factory.setConsumerFactory(cconsumerFactory());
factory.setConcurrency(concurrency);
factory.setBatchListener(true);
//SeekToCurrentErrorHandler eh = new SeekToCurrentErrorHandler((rec, ex) -> System.out.println("I am the dlq"), new ExponentialBackOff());
// factory.setBatchErrorHandler(errorHandler);*
return factory;
}
```

## Try Using Try-catch

- 에러 발생이 서버장애로 인한 것이라면 재시도를 통해 해당 비지니스를 정상적으로 진행시키는 것이 맞지만,
- 데이터 정합성 오류 또는 외부 API 호출 장애 등 재시도가 필요없을 경우는
- backoff기능 외에 subscribe하는 consumer클래스에서 try~catch로 잡는 방법이 있다.

## reference

[SeekToCurrentErrorHandler + ExponentialBackOff will log errors after backOff has passed, is that intentional?](https://stackoverflow.com/questions/64937593/seektocurrenterrorhandler-exponentialbackoff-will-log-errors-after-backoff-has)