---
layout: post
title: Error attempting to apply AttributeConverter
date: 2022-07-06 18:15:00
last_modified_at : 2022-07-06 18:15:00
parent: Errors
has_children: false
nav_exclude: true
tags: [jpa]
---

## Error attempting to apply AttributeConverter; nested exception is javax.persistence.PersistenceException: Error attempting to apply AttributeConverter

## error

```
10:40:52.851 40-exec-10 INFO  r.m.e.DevModeRestMessage: 19 process rest message is for development mode
10:40:52.852 40-exec-10 ERROR .ExceptionHandlerManager: 34 handleResponse  
[status] 500 INTERNAL_SERVER_ERROR, [code]: 500999, [message] Unexpected error occurred., 
[detail message] Error attempting to apply AttributeConverter; 
nested exception is javax.persistence.PersistenceException: Error attempting to apply AttributeConverter,
org.springframework.orm.jpa.JpaSystemException: 
Error attempting to apply AttributeConverter; 
nested exception is javax.persistence.PersistenceException: 
Error attempting to apply AttributeConverter
at org.springframework.orm.jpa.EntityManagerFactoryUtils.convertJpaAccessExceptionIfPossible(EntityManagerFactoryUtils.java:408)
at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible
(HibernateJpaDialect.java:257)
at org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.translateExceptionIfPossible(AbstractEntityManagerFactoryBean.java:531)

Caused by: CryptoException: 
com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'test': 
was expecting (JSON String, Number, Array, Object or token 'null', 'true' or 'false')
at [Source: (String)"test"; line: 1, column: 5]
at org.hibernate.metamodel.model.convert.internal.JpaAttributeConverterImpl.toRelationalValue(JpaAttributeConverterImpl.java:50)
at org.hibernate.type.descriptor.converter.AttributeConverterSqlTypeDescriptorAdapter$1.bind(AttributeConverterSqlTypeDescriptorAdapter.java:78)
... 161 common frames omitted
Caused by: com.fasterxml.jackson.core.JsonParseException: 
Unrecognized token 'test': was expecting (JSON String, Number, Array, Object or token 'null', 'true' or 'false')
at [Source: (String)"test"; line: 1, column: 5]
at com.fasterxml.jackson.core.JsonParser._constructError(JsonParser.java:1851)
at com.fasterxml.jackson.core.base.ParserMinimalBase._reportError(ParserMinimalBase.java:717)
at com.fasterxml.jackson.core.json.ReaderBasedJsonParser._reportInvalidToken(ReaderBasedJsonParser.java:2898)
at com.fasterxml.jackson.core.json.ReaderBasedJsonParser._reportInvalidToken(ReaderBasedJsonParser.java:2876)
at com.fasterxml.jackson.core.json.ReaderBasedJsonParser._matchToken2(ReaderBasedJsonParser.java:2673)
at com.fasterxml.jackson.core.json.ReaderBasedJsonParser._matchToken(ReaderBasedJsonParser.java:2651)
at com.fasterxml.jackson.core.json.ReaderBasedJsonParser._matchTrue(ReaderBasedJsonParser.java:2609)
at com.fasterxml.jackson.core.json.ReaderBasedJsonParser.nextToken(ReaderBasedJsonParser.java:741)
at com.fasterxml.jackson.databind.ObjectMapper._readTreeAndClose(ObjectMapper.java:4555)
at com.fasterxml.jackson.databind.ObjectMapper.readTree(ObjectMapper.java:2974)
... 166 common frames omitted
```

## cause

```java
@Lob
@Convert(converter = JsonCryptoConverter.class)
@Comment(value = "메시지 내용")
private String message;
```

converter를 지정했는데, json으로 읽어야하는 과정에서 일반 string으로 message를 넣어서 발생한 오류

converter 클래스부터 의심하자!