---
layout: post
title: Field doesn't have a default value
date: 2022-10-19 22:24:00
last_modified_at : 2024-06-13 08:08:00
parent: Errors
has_children: false
nav_exclude: true
tags: [sql]
---

## Problem
SQL Error: 1364, SQLState: HY000 Field doesn't have a default value

## Error

```prolog
[status] 500 INTERNAL_SERVER_ERROR, [code]: 500999, [message] Unexpected error occurred., 
[detail message] could not execute statement; 
nested exception is org.hibernate.exception.GenericJDBCException: 
could not execute statement, 
org.springframework.orm.jpa.JpaSystemException: could not execute statement; 
nested exception is org.hibernate.exception.GenericJDBCException: 
could not execute statement 
at org.springframework.orm.jpa.vendor.HibernateJpaDialect.convertHibernateAccessException(HibernateJpaDialect.java:331) 
at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:233) 
at org.springframework.orm.jpa.JpaTransactionManager.doCommit(JpaTransactionManager.java:566) 

SQL Error: 1364, SQLState: HY000

org.hibernate.engine.jdbc.spi.SqlExceptionHelper
Field 'c_registrable_f_r_user' doesn't have a default value
```

## cause

필드에 default값을 지정하지 않아서 발생

## try1.

찾아보니 sql mode를 변경하라고 해서 아래처럼 했는데도 오류는 같았습니다.

```sql
set session sql_mode = 'NO_ENGINE_SUBSTITUTION';
```

## try2.

위의 방식대로 해도 안되어서 계정의 권한을 보니 일부 권한이 누락되어있었습니다.

root계정으로 해당 계정에 권한을 모두 설정하니 잘 됩니다.

![sql_error_1364](../img/sql_error_1364.png)
