---
layout: post
title: Kafka Topic Change Notification with Slack
description: Kafka Topic Change Notification with Slack
date: 2023-07-13 16:10:00
last_modified_at : 2023-07-13 16:10:00
parent: Sub Projects
has_children: false
nav_exclude: true
keywords: kafka
---

# 목표

- 다른 어플리케이션에서 발행하는 토픽 정보가 바뀔때 대응개발이 필요한데, 코드를 보기 전까지 알기 어렵다.
- avro와 같은 것을 적용해서 스키마 관리를 하지 않기 때문에, 더더욱 변화를 알기 어려움
- 목표는 spring.json.type.mapping정보까지 diff하려고 한다.
    - avro를 사용하지 않으면, 해당 토픽을 consume해야한다.
- 일단 토픽명만 diff하는 것으로 프로젝트 구성

# 구성 환경

- 자바 1.8~
- spring boot 3.x
- kafka-client 2.6.x
- slack-api-client 1.29.2
- gradle 8.1.1

# 작업 내용

- kafka client 라이브러리를 통해 Admin API를 사용해서 topic 리스트를 조회한다.
    
    ```java
    Properties properties = new Properties();
    properties.put("bootstrap.servers", bootstrapServer);
    Admin admin = Admin.create(properties);
    Set<String> topicSet;
    try {
      topicSet = admin.listTopics().names().get();
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
    admin.close();
    return topicSet;
    ```
    
- 파일로 저장되어있는 토픽 리스트와 비교한다.
- 비교 후 차이가 발생한 토픽은 슬랙API를 사용해서 메시지 발송
    
    ```java
    static void publishMessage(String channelId, String message) {
        var client = Slack.getInstance().methods();
        try {
            var result = client.chatPostMessage(r -> r
                .token(token)
                .channel(channelId)
                .text(message)
            );
            log.info("result {}", result);
        } catch (IOException | SlackApiException e) {
            log.error("error: {}", e.getMessage(), e);
        }
    }
    ```

# Repository

# Snapshot

![kafka_topic_slack_notification.png](../img/kafka_topic_slack_notification.png)