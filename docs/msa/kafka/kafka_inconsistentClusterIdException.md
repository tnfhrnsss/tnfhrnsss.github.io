---
layout: post
title: kafka inconsistentClusterIdException
date: 2022-09-23 22:00:00
last_modified_at : 2022-09-23 22:00:00
parent: Kafka
grand_parent: Msa
nav_exclude: true
tags: [kafka, docker]
---

## problem

spring boot와 spring cloud를 버전업하면서

elasticsearch와 kafka 버전도 업그레이드를 해야했다.

kafka의 경우 2.6.0에서 3.1.1로 업그레이드를 했는데

기존에 docker에서 oss버전으로 쓰고 있던 것을 지원이 더이상 안되면서 bitnami로 바꿨다.

그러면서 아래 에러가 나면서 kakfka가 start 도중 shutdown 되었다.

## error log

```prolog
[2022-09-23 01:42:10,144] ERROR Fatal error during KafkaServer startup. 
Prepare to shutdown (kafka.server.KafkaServer)
kafka_1      | kafka.common.InconsistentClusterIdException: 
The Cluster ID Jpiu7Hx5RduFq5qGMf8zbQ doesn't match stored clusterId 
Some(Rkq4qbbhTIqEXFm19x6SiQ) in meta.properties. 
The broker is trying to join the wrong cluster. 
Configured zookeeper.connect may be wrong.
kafka_1      |  at kafka.server.KafkaServer.startup(KafkaServer.scala:228)
kafka_1      |  at kafka.Kafka$.main(Kafka.scala:109)
kafka_1      |  at kafka.Kafka.main(Kafka.scala)
kafka_1      |  INFO shutting down (kafka.server.KafkaServer)
```

## solve

방법은 의외로 간단했지만, 

구글링하면 window 환경의 docker기준이 아니라서 

meta.properties를 수정하라는 식의 해결방법 뿐이였다.

kafka가 올라가자마자 죽으니 서버에 접근도 불가능했기 때문에 방법이 없었는데..

1. Stop and Remove Kafka Containter
2. Remove Kafka Image
3. go to [Volumes] and deleta all of kafka data and log directories

![kafka_inconsistentClusterIdException](../img/kafka_inconsistentClusterIdException.png)