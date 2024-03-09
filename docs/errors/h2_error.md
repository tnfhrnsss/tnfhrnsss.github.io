---
layout: post
title: The write format 1 is smaller than the supported format 2
description: The write format 1 is smaller than the supported format 2
date: 2022-07-09 21:32:00
last_modified_at : 2022-12-10 18:46:00
parent: Errors
has_children: false
nav_exclude: true
tags: [h2]
---

## error

org.h2.jdbc.JdbcSQLNonTransientException
The write format 1 is smaller than the supported format 2

```
org.h2.jdbc.JdbcSQLNonTransientException: General error: 
"The write format 1 is smaller than the supported format 2 [2.0.206/5]" [50000-206]
at org.h2.message.DbException.getJdbcSQLException(DbException.java:573) ~[h2-2.0.206.jar:2.0.206]
at org.h2.message.DbException.getJdbcSQLException(DbException.java:496) ~[h2-2.0.206.jar:2.0.206]
at org.h2.message.DbException.get(DbException.java:216) ~[h2-2.0.206.jar:2.0.206]
at org.h2.mvstore.db.Store.convertMVStoreException(Store.java:166) ~[h2-2.0.206.jar:2.0.206]
at org.h2.mvstore.db.Store.<init>(Store.java:140) ~[h2-2.0.206.jar:2.0.206]
at org.h2.engine.Database.<init>(Database.java:324) ~[h2-2.0.206.jar:2.0.206]
at org.h2.engine.Engine.openSession(Engine.java:92) ~[h2-2.0.206.jar:2.0.206]
at org.h2.engine.Engine.openSession(Engine.java:222) ~[h2-2.0.206.jar:2.0.206]
at org.h2.engine.Engine.createSession(Engine.java:201) ~[h2-2.0.206.jar:2.0.206]
at org.h2.engine.SessionRemote.connectEmbeddedOrServer(SessionRemote.java:338) ~[h2-2.0.206.jar:2.0.206]
at org.h2.jdbc.JdbcConnection.<init>(JdbcConnection.java:117) ~[h2-2.0.206.jar:2.0.206]
at org.h2.Driver.connect(Driver.java:59) ~[h2-2.0.206.jar:2.0.206]
at com.zaxxer.hikari.util.DriverDataSource.getConnection(DriverDataSource.java:138) ~[HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.pool.PoolBase.newConnection(PoolBase.java:358) ~[HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.pool.PoolBase.newPoolEntry(PoolBase.java:206) ~[HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.pool.HikariPool.createPoolEntry(HikariPool.java:477) [HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.pool.HikariPool.checkFailFast(HikariPool.java:560) [HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.pool.HikariPool.<init>(HikariPool.java:115) [HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.HikariDataSource.getConnection(HikariDataSource.java:112) [HikariCP-3.4.5.jar:na]
at org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration.lambda$h2Console$0(H2ConsoleAutoConfiguration.java:72) [spring-boot-autoconfigure-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.beans.factory.ObjectProvider.ifAvailable(ObjectProvider.java:93) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration.h2Console(H2ConsoleAutoConfiguration.java:71) [spring-boot-autoconfigure-2.3.12.RELEASE.jar:2.3.12.RELEASE]
```

## solved

h2버전 업 전에 생성했던 h2파일이 이전 버전이어서 생긴 문제로 보여서

Delete the old version of h2 files and restart h2 service.