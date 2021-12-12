---
layout: default
title: Timeout of 60000ms expired before the position...
parent: Errors
has_children: false
nav_exclude: true
---

# kafdrop 보려는데 Timeout of 60000ms expired before the position for partition retry-assignment-2 could be determined 에러날 때

현상) kafdrop에 접속해서 스트림보려는데, 에러날때

애러)

## Internal Server Error

A 500 error has occurred: Request processing failed; nested exception is [org.apache.kafka.common.errors.TimeoutException](https://www.blogger.com/blog/post/edit/2689228726924373128/7388999278163586944#): Timeout of 60000ms expired before the position for partition retry-assignment-2 could be deter

원인) host에 주소가 잘못박혀있거나.. 디폴트(60000ms) 로딩시간을 초과했다거나 할때 발생

해결) 타임아웃 늘리려면, ProducerConfig.MAX_BLOCK_MS_CONFIG 설정을 주면 된다.

아니면 도커에 컨테이너 삭제하고 다시 deploy

[참고]

[stackoverflow.com/a/60068634/14257397](https://www.blogger.com/blog/post/edit/2689228726924373128/7388999278163586944#)

[org.apache.kafka.common.errors.TimeoutException
I have two broker 1.0.0 kafka cluster and I am running 1.0.0 kafka stream API application against this kafka.I increased the producer request.timeout.ms to 5 minutes to fix producer Timeoutexceptio...
stackoverflow.com](https://www.blogger.com/blog/post/edit/2689228726924373128/7388999278163586944#)