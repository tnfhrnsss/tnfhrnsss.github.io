---
layout: post
title: Timeout of 60000ms expired before the position...
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
tags: [kafka, kafdrop]
---

### kafdrop 보려는데 Timeout of 60000ms expired before the position for partition retry-assignment-2 could be determined 에러날 때

#### problem
kafdrop에 접속해서 스트림보려는데, 에러날때

#### error log

##### Internal Server Error

```
A 500 error has occurred: Request processing failed; nested exception is 
Timeout of 60000ms expired before the position for partition retry-assignment-2 could be deter
```

#### cause
host에 주소가 잘못박혀있거나.. 디폴트(60000ms) 로딩시간을 초과했다거나 할때 발생

#### solution
타임아웃 늘리려면, ProducerConfig.MAX_BLOCK_MS_CONFIG 설정을 주면 된다.

아니면 도커에 컨테이너 삭제하고 다시 deploy

#### reference

- [stackoverflow.com/a/60068634/14257397](https://www.blogger.com/blog/post/edit/2689228726924373128/7388999278163586944#)

- [org.apache.kafka.common.errors.TimeoutException](https://www.blogger.com/blog/post/edit/2689228726924373128/7388999278163586944#)