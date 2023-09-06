---
layout: post
title: org.springframework.kafka.KafkaException
date: 2021-09-09 13:07:45
last_modified_at : 2023-09-07 07:42:00
parent: Errors
has_children: false
nav_exclude: true
tags: [kafka]
---

### org.springframework.kafka.KafkaException: Ambiguous methods for payload type

#### error log

```
org.springframework.kafka.KafkaException: Ambiguous methods for payload type:
** class MessageCreated: handle and handle
...
```


#### cause

```
@KafkaListener(
...
)
public class EventPushSubscriber {
    
    @KafkaHandler
    public void handle(Object object) {
        ...
    }

    @KafkaHandler
    public void handle(MessageCreated events) {
        ...
    }
}

```
내가 KafkaHandler의 메소드 인자를 Object타입으로 받은거랑 MessageCreated로 받은걸 두개 생성해서... 어디에 매핑할지 모호하다는 에러임. Object인자로 받는 메소드를 지우자