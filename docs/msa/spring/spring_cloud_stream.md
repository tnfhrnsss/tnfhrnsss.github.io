---
layout: post
title: spring-cloud-stream's kafka vs spring-kafka
description: spring-cloud-stream's kafka vs spring-kafka
date: 2023-08-24 08:23:00
last_modified_at : 2023-08-24 08:23:00
parent: Spring
grand_parent: Msa
nav_exclude: true
tags: [spring cloud, kafka]
---

# spring-cloud-stream?

- Spring Cloud Stream은 스프링 기반의 마이크로서비스 아키텍처를 지원하기 위한 프로젝트 중 하나로
- 스프링 애플리케이션과 메시징 미들웨어(Messaging Middleware) 사이의 통합을 단순화하는 데 중점을 두고 있다.
- 그래서 Spring Cloud Stream을 사용하면 애플리케이션 간의 메시지 기반 커뮤니케이션을 쉽게 구현할 수 있다.

### Spring Cloud Stream 주요 기능

1. Binder Abstraction 제공
    1. 메시징 미들웨어 (예: Apache Kafka, RabbitMQ, Amazon Kinesis 등)와 통합할 수 있는 바인더 추상화를 제공한다.
    2. 이를 통해 애플리케이션 코드를 거의 수정하지 않고도 다른 메시징 시스템으로 전환할 수 있다.
2. Message Driven MSA
    1. 메시지 기반 아키텍처를 쉽게 구현할 수 있도록 지원
    2. 어플리케이션 간의 통신은 메시지를 통해 이루어지며, 각 컴포넌트는 메시지를 받아서 필요한 비지니스를 처리한다.
3. 바인딩(Binding)
    1. Spring Cloud Stream은 입력과 출력 채널 간의 바인딩을 지원한다. 
4. 다양한 메시징 패턴 지원
    1. 메시지 라우팅, 메시지 변환, 메시지 그룹화 등 다양한 메시징 패턴을 지원한다.
5. 메시지 프로토콜 지원
    1. 다양한 메시지 프로토콜을 지원하여 데이터 직렬화 및 역직렬화를 쉽게 처리할 수 있다.
6. 기타
    1. Spring Cloud Stream은 Spring Integration 프로젝트와 결합하여 기능을 확장할 수도 있다.
    2. 높은 수준의 추상화와 간편한 설정을 통해 메시징 시스템을 사용할 수 있게 한다.

# spring-cloud-stream 내에서의 kafka와 spring-kafka와의 차이는?

Spring Cloud Stream과 Spring Kafka는 둘 다 스프링 프레임워크 기반의 메시징 솔루션으로 목적과 사용 방법에서 차이가 있다.

### Spring Kafka:

- Spring Kafka는 스프링 프레임워크와 Apache Kafka를 통합하기 위한 라이브러리
- Kafka를 직접 사용하기 위한 기능들을 제공하며, 주로 Kafka 클라이언트 라이브러리를 사용하여 직접 Kafka 클러스터와 통신하는 방식으로 동작한다.
- Spring Kafka를 사용하면 간단하게 Kafka Producer 및 Consumer를 만들고, 메시지를 전송하고 수신하는 작업을 수행할 수 있다.
- 또한 Kafka의 고급 기능들을 노출하고 조정하는 데 유용합니다.

### Spring Cloud Stream:

- Spring Cloud Stream은 Spring Integration 프로젝트를 기반으로 구축되어 메시징 시스템과의 통합을 단순화하는 추상화 계층을 제공한다.
- Spring Cloud Stream은 메시지 기반 마이크로서비스 아키텍처를 지원한다.
- Kafka 뿐만 아니라 RabbitMQ, Amazon Kinesis 등 다양한 메시징 시스템과 통합할 수 있고, 런타임에 사용하는 메시징 미들웨어를 쉽게 교체할 수 있다.
- Spring Cloud Stream은 라우팅, 변환, 그룹화 등 다양한 메시징 패턴과 메시지 처리 기능을 제공.

# Summary

- Spring Kafka는 주로 Kafka 클라이언트 라이브러리를 사용하여 Kafka와 직접 상호 작용하고자 할 때 사용되며,
- Spring Cloud Stream은 메시징 미들웨어와의 통합을 간소화하여 더 추상화된 방식으로 메시지 기반 마이크로서비스를 개발하고자 할 때 사용된다.
- Spring Cloud Stream은 Spring Kafka를 기반으로 구현될 수도 있으며, Spring Kafka를 사용하는 경우 Spring Cloud Stream의 일부 기능을 사용할 수도 있다.