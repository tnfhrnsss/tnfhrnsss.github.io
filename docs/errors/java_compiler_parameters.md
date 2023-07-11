---
layout: post
title: java compiler 매개변수 이름을 유지할 때 발생하는 deserializer 문제
description: java compiler 매개변수 이름을 유지할 때 발생하는 deserializer 문제
date: 2023-07-11 14:29:00
last_modified_at : 2023-07-11 14:29:00
parent: Errors
has_children: false
nav_exclude: true
keywords: javac
---

# Error

```java
Caused by: com.fasterxml.jackson.databind.exc.MismatchedInputException: 
Input mismatch reading Enum `domain.enums.BusinessChannelType`: 
properties-based `@JsonCreator` ([method domain.enums.BusinessChannelType#of(java.lang.String)]) 
expects JSON Object (JsonToken.START_OBJECT), got JsonToken.VALUE_STRING
 at [Source: (org.springframework.util.StreamUtils$NonClosingInputStream); line: 5, column: 29] (through reference chain:)
	at com.fasterxml.jackson.databind.exc.MismatchedInputException.from(MismatchedInputException.java:59)
	at com.fasterxml.jackson.databind.DeserializationContext.reportInputMismatch(DeserializationContext.java:1741)
	at com.fasterxml.jackson.databind.deser.std.FactoryBasedEnumDeserializer.deserialize(FactoryBasedEnumDeserializer.java:137)
	at com.fasterxml.jackson.databind.deser.impl.FieldProperty.deserializeAndSet(FieldProperty.java:138)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserializeFromObject(BeanDeserializer.java:392)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:185)
	at com.fasterxml.jackson.databind.deser.impl.FieldProperty.deserializeAndSet(FieldProperty.java:138)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserializeFromObject(BeanDeserializer.java:392)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:185)
	at com.fasterxml.jackson.databind.deser.DefaultDeserializationContext.readRootValue(DefaultDeserializationContext.java:323)
```

# Cause

- 컴파일러에게 매개변수 이름을 사용하기 위해 compilerArgs옵션 추가된것이 역직렬화 문제 발생시키게 된 것

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>${maven-compiler-plugin.version}</version>
    <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
        <encoding>${project.build.sourceEncoding}</encoding>
        <compilerArgs>
            <arg>-parameters</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

## **-parameters 옵션?**

- 컴파일러에게 메서드의 매개변수 이름을 유지하도록 하는 것으로, 이 옵션을 추가하면  Java Reflection을 사용하여 메서드 매개변수 이름을 추출할 수 있다. 그래서 클래스 파일에 var~로 시작하는 것 대신 java파일의 변수명으로 알 수 있게 된다.
- 이는 주로 런타임에서 리플렉션을 사용하는 프레임워크인 Spring Framework에서 매개변수 이름에 기반한 작업을 수행할 때 유용하다.

## parameters옵션과 역직렬화 문제

- 근데 이 옵션을 사용하면 Jackson 또는 다른 JSONParser에서 Enum 클래스를 역직렬화할 때 문제가 발생하게 된다.
- JSON 파서는 Enum 값을 식별하기 위해 Enum 상수의 이름을 사용하는 경우가 많은데,  **`-parameters`** 옵션을 사용하면 컴파일러가 메서드 매개변수의 이름을 변경하므로, JSON 파서는 예상한 Enum 상수 이름과 다른 이름을 발견하게 되고, 이로 인해 "JSON parse error: Input mismatch reading Enum"과 같은 오류가 발생하는 것

# Solve

## Try1. **`@JsonProperty` 사용**

- 해당 enum 클래스에서 @JsonCreator 어노테이션을 사용할 때 **`@JsonProperty`** 어노테이션과 함께 선언하도록 한다.
    
    ```java
    public enum MyEnum {
        @JsonProperty("VALUE_ONE")
        VALUE1,
    
        @JsonProperty("VALUE_TWO")
        VALUE2;
    
        @JsonCreator
        public static MyEnum fromJson(String value) {
            // 역직렬화 로직
        }
    }
    ```
    

## Try2. **`@lombok.experimental.NonFinal` 사용**

- Lombok 라이브러리를 사용하면 **`@lombok.experimental.NonFinal`** 어노테이션을 Enum 클래스에 추가하여 메서드 매개변수 이름을 유지할 수 있다고 한다. (해보지 않음)
    
    ```java
    import lombok.experimental.NonFinal;
    
    public enum MyEnum {
        @NonFinal
        VALUE1,
    
        @NonFinal
        VALUE2;
        
        // 역직렬화 로직
    }
    ```
    

## Try3. `JsonCreator.Mode.*DELEGATING`* 사용

- **`@JsonCreator`** 어노테이션을 사용하여 Enum 클래스의 생성자에 **`(mode = JsonCreator.Mode.DELEGATING)`** 옵션을 추가하면 해당 Enum 클래스를 역직렬화할 때 발생하는 문제를 해결할 수 있다.
- 이 모드는 생성자 매개변수의 순서와 관계없이 매개변수 이름을 기반으로 역직렬화를 수행한다.
    
    ```java
    @Getter
    @AllArgsConstructor
    public enum BusinessChannelType implements CamelableEnum {
        A_TALK("a.talk"),
        B_TALK("b.talk")
        ;
    
        private final String value;
    
        @JsonCreator(mode = JsonCreator.Mode.DELEGATING)
        public static BusinessChannelType of(String value) {
            return BusinessChannelType.valueOf(CamelcaseUtil.toSnakeName(value));
        }
    
        @JsonValue
        @Override
        public String camelName() {
            return CamelcaseUtil.toCamelName(name());
        }
    }
    ```