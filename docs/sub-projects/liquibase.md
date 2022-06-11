---
layout: post
title: Liquibase
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Sub Projects
has_children: false
nav_order: 5
nav_exclude: true
---

# Liquibase

검토 배경

- 사이트 마이그레이션을 용이하게 하려면?
- 수동으로 쿼리를 작성하게 되면, 누락되는 항목도 생기고, 버그도 발생

liquibase 주요 기능

- change log
- tracking work

검토 결과

Q1. data diff가 가능한가?

- json data가 저장된 컬럼 비교가 가능한가?

A1. 

- 자동으로 비교할 수 없고, liquibase db table에 변경이력을 쌓아야만 이력추적을 통해 가능
- 그 외에 다른 database object들에 대한 diff는 이력으로 쌓지 않아도 모두 가능

```bash
./liquibase --outputFile=mydiff.txt --driver=com.mysql.jdbc.Driver 
--classpath=lib/mysql-connector-java-8.0.25.jar 
--url="jdbc:mysql://localhost:3306/db1?serverTimezone=Asia/Seoul&characterEncoding=UTF-8" 
--username=root --password=password 
--referenceUrl="jdbc:mysql://localhost:3306/db2?serverTimezone=Asia/Seoul&characterEncoding=UTF-8" 
--referenceUsername=root --referencePassword=password -diffTypes=data diff
```

```
[mydiff.txt]
Reference Database: root@localhost @ jdbc:mysql://localhost:3306/db2?serverTimezone=Asia/Seoul&characterEncoding=UTF-8 (Default Schema: db2)
Comparison Database: root@localhost @ jdbc:mysql://localhost:3306/db1?serverTimezone=Asia/Seoul&characterEncoding=UTF-8 (Default Schema: db1)
Compared Schemas: db2 -> db1
Product Name: EQUAL
Product Version: EQUAL
Missing Catalog(s): NONE
Unexpected Catalog(s): NONE
Changed Catalog(s): NONE
Missing Column(s): 
     db2.mocha_survey_questionnaire.C_ACTIVE
     db2.mocha_survey.C_MESSAGE
Unexpected Column(s): 
     db1.mocha_knowledge.C_ACCESS_TYPE
     db1.mocha_removed_team.C_ALIAS
Changed Column(s): 
     db2.mocha_survey_response.C_CREATED_AT
          order changed from '5' to '4'
     db2.mocha_survey.C_ENABLED
          nullable changed from 'false' to 'true'
          order changed from '6' to '8'
Missing Foreign Key(s): NONE
Unexpected Foreign Key(s): NONE
Changed Foreign Key(s): NONE
Missing Index(s): NONE
Unexpected Index(s): NONE
Changed Index(s): NONE
Missing Primary Key(s): NONE
Unexpected Primary Key(s): NONE
Changed Primary Key(s): NONE
Missing Sequence(s): NONE
Unexpected Sequence(s): NONE
Changed Sequence(s): NONE
Missing Table(s): 
     mocha_survey_questionnaire
     mocha_survey_restrict
Unexpected Table(s): 
     crema_active_kakao
Changed Table(s): NONE
Missing Unique Constraint(s): NONE
Unexpected Unique Constraint(s): NONE
Changed Unique Constraint(s): NONE
Missing View(s): NONE
Unexpected View(s): NONE
Changed View(s): NONE
```

활용 방법

- 제품의 버전별 DB이력관리가 필요하다면, git log를 변경이력으로 등록하게 해서 활용하는 방법을 생각해볼 수 있다.