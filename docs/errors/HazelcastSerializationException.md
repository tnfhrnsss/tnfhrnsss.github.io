---
layout: post
title: HazelcastSerializationException
date: 2022-04-15 23:30:00
last_modified_at : 2022-04-15 23:30:00
parent: Errors
has_children: false
nav_exclude: true
tags: [hazelcast, jsonserialization]
---

### error

```
nested exception is com.hazelcast.nio.serialization.HazelcastSerializationException: 
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field "btalkId", 
not marked as ignorable (one known property: "customerId"])

at [Source: (String)"{

"customerId" : "DS3l1Bl_RGz2",

"btalkId" : "0edc1246-cb45-4f87-bdc0-9768890c6614"

}"; line: 3, column: 16] (through reference chain: BTalk["btalkId"]);
```

### cause

```
public class BtalkKey implements JsonSerializable {    
	private String bTalkId;
}
```

변수를 bTalkId; 로 선언했더니, getter에서 getBTalkId로 되었지만, Hazelcast에서 json으로 찾으려고 할때, btalkId로 찾아서 발생한 오류

### resolve

자바표준은 getter로 썼을때, 대문자가 연속으로 두번이 오는 것은 표준에 어긋난다고 함. 

그래서 bTalkId 에서 btalkId;로 변수 선언

### reference

[why-does-jackson-2-not-recognize-the-first-capital-letter-if-the-leading-camel-c](https://stackoverflow.com/questions/30205006/why-does-jackson-2-not-recognize-the-first-capital-letter-if-the-leading-camel-c)

[java-tip-6-dont-capitalize-first-two](http://futuretask.blogspot.com/2005/01/java-tip-6-dont-capitalize-first-two.html)