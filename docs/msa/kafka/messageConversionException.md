---
layout: post
title: MessageConversionException failed to resolve class name
date: 2023-11-05 22:39:00
last_modified_at : 2023-11-05 22:39:00
parent: Kafka
grand_parent: Msa
nav_exclude: true
tags: [kafka, MessageConversionException]
description: MessageConversionException failed to resolve class name
---

# error log

```java
11:27:42.751 ERROR [,] [r#10-0-C-1] m.c.s.KafkaFailedDeserializationFunction: 
12 apply           topic [event.test]
org.springframework.messaging.converter.MessageConversionException: 
failed to resolve class name. Class not found [test.com.TestService]; 
nested exception is java.lang.ClassNotFoundException: test.com.TestService
	at org.springframework.kafka.support.mapping.DefaultJackson2JavaTypeMapper.getClassIdType(DefaultJackson2JavaTypeMapper.java:142)
	at org.springframework.kafka.support.mapping.DefaultJackson2JavaTypeMapper.toJavaType(DefaultJackson2JavaTypeMapper.java:103)
	at org.springframework.kafka.support.serializer.JsonDeserializer.deserialize(JsonDeserializer.java:572)
```

# cause

- [kafka_serializationException/#solved4](https://tnfhrnsss.github.io/docs/msa/kafka/kafka_serializationException/#solved4) 방식처럼 spring.json.type.mapping를 통해서 메시지 역직렬화 클래스를 지정할 때, 해당 topic으로 전달되는 모든 json데이터를 정의하지 않으면, MessageConversionException가 발생하게 된다.
- 하지만 해당 토픽에서 발생하는 모든 이벤트 클래스를 맵핑하는 것에 대해 비지니스 처리가 필요하지 않는다면 전부를 맵핑용 클래스로 생성할 필요가 없기 때문에 방법이 필요

# solve 1.

- Object클래스로 맵핑을 시키고, 해당 클래스로 캐스팅하는 방법이 있다.

```java
@KafkaListener(
    id = CtalkAppChannelEventSubscriber.ID,
    topics = KafkaTenant.PREFIX + CtalkTopics.EVENT_APP_CHANNEL,
    groupId = CtalkTopics.EVENT_APP_CHANNEL + "-messengerconnector-ctalk-appsetting-subscriber",
    properties = {
        "spring.json.type.mapping:" +
            "test.com.messaging.event.MessageCreated:java.lang.Object," +
            "test.com.messaging.event.TestAService:test.com.TestAService," +
            "test.com.messaging.event.TestBService:test.com.TestBService"
    },
    autoStartup = "false"
)
```

```java
@KafkaHandler
public void handle(Object object) {
    try {
        if (object instanceof MessageCreated) {
            log.debug("MessageCreated event");
        } else if (object instanceof MessageDeleted) {
            log.debug("MessageDeleted event");
        } else {
            log.debug("else!!!");
        }
    } catch (Exception e) {
       
    }
}

@KafkaHandler
public void handle(TestAService testAService) {
    try {
        // TestAService 
    } catch (Exception e) {
       
    }
}
```

- 위의 방법은 Object로 맵핑시킬 class path는 일치해야하는 단점이 있다.
- `DefaultJackson2JavaTypeMapper` 에서 일치하지 않으면, `MessageConversionException` throw한다.

```sql
private JavaType getClassIdType(String classId) {
        if (this.getIdClassMapping().containsKey(classId)) {
            return TypeFactory.defaultInstance().constructType((Type)this.getIdClassMapping().get(classId));
        } else {
            try {
                if (!this.isTrustedPackage(classId)) {
                    throw new IllegalArgumentException("The class '" + classId + "' is not in the trusted packages: " + this.trustedPackages + ". If you believe this class is safe to deserialize, please provide its name. If the serialization is only done by a trusted source, you can also enable trust all (*).");
                } else {
                    return TypeFactory.defaultInstance().constructType(ClassUtils.forName(classId, this.getClassLoader()));
                }
            } catch (ClassNotFoundException var3) {
                throw new MessageConversionException("failed to resolve class name. Class not found [" + classId + "]", var3);
            } catch (LinkageError var4) {
                throw new MessageConversionException("failed to resolve class name. Linkage error [" + classId + "]", var4);
            }
        }
    }
```

- 그래서 이 부분은 어쩔 수 없지만, 실제로 클래스를 생성하지 않아도 되니 일단 이 방법으로 적용했다.

# better than solve1.

- Object말고 별도로 제네릭 타입을 받도록 디폴트 클래스를 생성해서 처리할 수 있다.

```java
@KafkaListener(
    id = CtalkAppChannelEventSubscriber.ID,
    topics = KafkaTenant.PREFIX + CtalkTopics.EVENT_APP_CHANNEL,
    groupId = CtalkTopics.EVENT_APP_CHANNEL + "-messengerconnector-ctalk-appsetting-subscriber",
    properties = {
        "spring.json.type.mapping:" +
            "test.com.messaging.event.MessageCreated:test.com.Ignore," +
            "test.com.messaging.event.TestAService:test.com.TestAService," +
            "test.com.messaging.event.TestBService:test.com.TestBService"
    },
    autoStartup = "false"
)
public class CtalkAppChannelEventSubscriber {
	@KafkaHandler
	public void handle(TestAService testAService) {
	    try {
	        // TestAService 
	    } catch (Exception e) {
	       
	    }
	}
	
	@KafkaHandler(isDefault = true)
	public void ignore(Ignore ignore) {
	    log.trace("Received ignore: {}", ignore);
	}
}
```