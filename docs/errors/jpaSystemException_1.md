---
layout: post
title: SQLException Incorrect string value '\xA4\xB7'
date: 2023-05-27 16:11:00
last_modified_at : 2023-05-27 16:11:00
parent: Errors
has_children: false
nav_exclude: true
---

# Problem

```prolog
10:16:02.303 040-exec-6 ERROR .ExceptionHandlerManager: 34 handleResponse  [status] 500 INTERNAL_SERVER_ERROR, [code]: 500999, [message] Unexpected error occurred., [detail message] could not execute statement; nested exception is org.hibernate.exception.GenericJDBCException: could not execute statement, [module] mocha
org.springframework.orm.jpa.JpaSystemException: could not execute statement; nested exception is org.hibernate.exception.GenericJDBCException: could not execute statement
	at org.springframework.orm.jpa.vendor.HibernateJpaDialect.convertHibernateAccessException(HibernateJpaDialect.java:331)
	at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:233)
	at org.springframework.orm.jpa.JpaTransactionManager.doCommit(JpaTransactionManager.java:566)
	at org.springframework.transaction.support.AbstractPlatformTransactionManager.processCommit(AbstractPlatformTransactionManager.java:743)
	at org.springframework.transaction.support.AbstractPlatformTransactionManager.commit(AbstractPlatformTransactionManager.java:711)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.commitTransactionAfterReturning(TransactionAspectSupport.java:654)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:407)
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119)
	...
Caused by: java.sql.SQLException: Incorrect string value: '\xA4\xB7","m...' for column 'c_message' at row 1
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:129)
	at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:122)
	at com.mysql.cj.jdbc.ClientPreparedStatement.executeInternal(ClientPreparedStatement.java:916)
	at com.mysql.cj.jdbc.ClientPreparedStatement.executeUpdateInternal(ClientPreparedStatement.java:1061)
	at com.mysql.cj.jdbc.ClientPreparedStatement.executeUpdateInternal(ClientPreparedStatement.java:1009)
	at com.mysql.cj.jdbc.ClientPreparedStatement.executeLargeUpdate(ClientPreparedStatement.java:1320)
	at com.mysql.cj.jdbc.ClientPreparedStatement.executeUpdate(ClientPreparedStatement.java:994)
	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)
	at com.zaxxer.hikari.pool.HikariProxyPreparedStatement.executeUpdate(HikariProxyPreparedStatement.java)
	at org.hibernate.engine.jdbc.internal.ResultSetReturnImpl.executeUpdate(ResultSetReturnImpl.java:197)
	... 149 common frames omitted
```

# Cause
- tablespace 캐릭터셋을 utf8로 생성해서 제대로 인입되지 않는 현상으로 utf8mb4로 변경
- mysql8버전부터 기본 캐릭터셋은 utf8mb4과 utf8mb4_0900_ai_ci이므로 별도 설정 불필요.

# Reference
- https://www.lesstif.com/dbms/mysql-rhel-centos-ubuntu-20775198.html