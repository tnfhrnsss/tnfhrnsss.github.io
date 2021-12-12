---
layout: default
title: current transaction is aborted, commands ignored until end of transaction block
parent: Errors
has_children: false
nav_exclude: true
---

# current transaction is aborted, commands ignored until end of transaction block

[현상]

postgresql - jdbc로 커넥션 맺어서 자바에서 ibatis쿼리 실행하는데 아래와 같은 에러가 났다.

**current transaction is aborted, commands ignored until end of transaction block**

- -- The error occurred while applying a parameter map.
- -- Check the .
    
    **selectNextOne**
    
- -- Check the statement (update procedure failed).
- **-- Cause: org.postgresql.util.PSQLException: ERROR: current transaction is aborted, commands ignored until end of transaction block**

at com.ibatis.sqlmap.engine.mapping.statement.MappedStatement.executeQueryWithCallback(MappedStatement.java:201)

at com.ibatis.sqlmap.engine.mapping.statement.MappedStatement.executeQueryForObject(MappedStatement.java:120)

at com.ibatis.sqlmap.engine.impl.SqlMapExecutorDelegate.queryForObject(SqlMapExecutorDelegate.java:527)

at com.ibatis.sqlmap.engine.impl.SqlMapExecutorDelegate.queryForObject(SqlMapExecutorDelegate.java:502)

at com.ibatis.sqlmap.engine.impl.SqlMapSessionImpl.queryForObject(SqlMapSessionImpl.java:106)

at org.springframework.orm.ibatis.SqlMapClientTemplate$1.doInSqlMapClient(SqlMapClientTemplate.java:270)

at org.springframework.orm.ibatis.SqlMapClientTemplate.execute(SqlMapClientTemplate.java:200)

... 52 more

[원인]

위에 처럼 나오면 쿼리에 문법적 오류로 인하여 현재 트랜잭션은 폐기된다는 뜻

내 경우는 selectNextOne쿼리가 잘못되었음