---
layout: post
title: docker elasticsearch shutdown cause
description: docker elasticsearch shutdown cause
date: 2022-09-21 21:28:00
last_modified_at : 2022-09-21 21:28:00
parent: Errors
has_children: false
nav_exclude: true
tags: [docker, elasticsearch]
keywords: docker
---

## problem 1

```java
ERROR: [2] bootstrap checks failed. You must address the points described in the following [2] lines before starting Elasticsearch.

bootstrap check failure [1] of [2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

bootstrap check failure [2] of [2]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
```

## solved

window의 m.max_map_count는 cmd창 열어서

```
1) wsl -d docker-desktop
2) sysctl -w vm.max_map_count=262144
```

[ElasticSearch in Windows docker image vm max map count](https://stackoverflow.com/a/63882309)

## problem 2

```python
the default discovery settings are unsuitable for production use; 
at least one of [discovery.seed_hosts, discovery.seed_providers, 
cluster.initial_master_nodes] must be configured
```

docker-compose.yml에 아래 추가

```java
environment:
  discovery.seed_hosts: "elasticsearch"
  cluster.initial_master_nodes: "node"
```

[Elasticsearch 설치 시 오류](https://horae.tistory.com/entry/Elasticsearch-%EC%84%A4%EC%B9%98-%EC%8B%9C-%EC%98%A4%EB%A5%98-the-default-discovery-settings-are-unsuitable-for-production-use-at-least-one-of-discoveryseedhosts-discoveryseedproviders-clusterinitialmasternodes-must-be-configured)  