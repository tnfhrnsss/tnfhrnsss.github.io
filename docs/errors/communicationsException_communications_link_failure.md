---
layout: post
title: CommunicationsException Communications link failure
description: CommunicationsException Communications link failure
date: 2024-04-19 23:42:00
last_modified_at : 2024-04-19 23:42:00
parent: Errors
has_children: false
nav_exclude: true
---

# Problem

- spring boot를 시작하면 
com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link failure에러가 발생하면서 올라가지 않는다.

# Error

```java
07:51:43.022 ERROR [,] [      main] com.zaxxer.hikari.pool.HikariPool       
:594 zationException hikaricp-pool - Exception during pool initialization.
com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago. 
The driver has not received any packets from the server.
	at com.mysql.cj.jdbc.exceptions.SQLError.createCommunicationsException(SQLError.java:174)
	at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:64)
	at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:815)
	at com.mysql.cj.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:438)
```

# Cause

- remote 서버의 db를 접속해서 사용하는 환경인데, remote서비스에 설정한 datasource.url이 localhost로 작성되어있어서 그렇다.

```yaml
spring:
  datasource:
    url: jdbc:mysql://172.xxx.xxx.xxx:3306/test?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
```

# Reference

- [https://mr-docker.tistory.com/6](https://mr-docker.tistory.com/6)