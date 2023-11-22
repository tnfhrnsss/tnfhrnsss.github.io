---
layout: post
title: Add apm in kibana
date: 2022-11-15 21:07:00
last_modified_at : 2022-11-15 21:07:00
parent: Monitoring
grand_parent: Quality Practice
nav_exclude: true
description: Add apm in kibana
---

# 설치 버전 정보

- Elasticsearch version 7.17.6
- Kibana 7.6.2
- apm-server-7.6.2-x86_64.rpm

# APM 설치

## 1. server 설치

- 사전 준비
    - es와 kibana는 설치가 되어있다고 가정

### Linux
- 순서
    - kibana 콘솔(localhost:5601)에서 APM 서버 add 선택하면 상세 순서가 나옵니다.
    - rpm 다운 및 설치
        
        ```prolog
        curl -L -O https://artifacts.elastic.co/downloads/apm-server/apm-server-7.6.2-x86_64.rpm
        sudo rpm -vi apm-server-7.6.2-x86_64.rpm
        ```
        
    - root 권한 계정으로 서비스 시작
        
        ```prolog
        service apm-server start
        ```
        
    - APM 서비스 상태 확인
        - kibana화면에서 ([http://localhost:5601/app/kibana#/home/tutorial/apm](http://172.16.120.183:5601/app/kibana#/home/tutorial/apm)) check
        
        ![apm_kibana.png](../img/apm_kibana.png)

### Window
- 순서
    - [APM 다운로드 페이지](https://www.elastic.co/kr/downloads/apm) 에서 다운로드받는다.
    - 압축 풀고
    - elasticsearch 서버가 로컬과 다르다면 apm-server.yml 수정한다. (디폴트는 localhost:9200)
    - start the APM Server

        ```
        ./apm-server -e
        ```
        

## 2. APM Agents 설치

- [https://search.maven.org/search?q=a:elastic-apm-agent](https://search.maven.org/search?q=a:elastic-apm-agent) 에서 jar로 다운받고
- 서비스 실행 javaopt에 추가
    
    ```bash
    java -javaagent:/home/apm/elastic-apm-agent-1.34.1.jar \
    -Delastic.apm.service_name=victory \
    -Delastic.apm.server_urls=http://localhost:8200 \
    ```
    
- apm agent 서비스 상태 확인
    - kibana화면에서 체크
        
        ![apm_kibana1.png](../img/apm_kibana1.png)
        
- Load Kibana objects 실행

# Monitoring

- JVM, Transaction, Error를 어플리케이션별로 조회할 수 있고,
- 느린 쿼리 및 스택정보도 확인이 가능합니다…..최고b

![apm_kibana2.png](../img/apm_kibana2.png)

![apm_kibana3.png](../img/apm_kibana3.png)

## Error1

apm rpm 설치 후 service apm-server start했는데 아래와 같은 에러가 messages에 찍히고 있고 elasticsearch와 연결되지 않음

```prolog
2022-11-14T10:29:30.357+0900#011ERROR#011elasticsearch/client.go:350#011
Failed to perform any bulk index operations: 500 Internal Server Error: 
{"error":{"root_cause":[{"type":"illegal_state_exception",
"reason":"There are no ingest nodes in this cluster, 
unable to forward request to an ingest node."}],
"type":"illegal_state_exception",
"reason":"There are no ingest nodes in this cluster, 
unable to forward request to an ingest node."},"status":500}
2022-11-14T10:29:32.019+0900#011ERROR#011pipeline/output.go:121#011
Failed to publish events: 500 Internal Server Error: 
{"error":{"root_cause":[{"type":"illegal_state_exception",
"reason":"There are no ingest nodes in this cluster, 
unable to forward request to an ingest node."}],
"type":"illegal_state_exception",
"reason":"There are no ingest nodes in this cluster, 
unable to forward request to an ingest node."},"status":500}
2022-11-14T10:29:32.019+0900#011INFO#011pipeline/output.go:95#011Connecting to backoff(elasticsearch(http://localhost:9200))
2022-11-14T10:29:32.024+0900#011INFO#011elasticsearch/client.go:757#011Attempting to connect to Elasticsearch version 7.17.6
2022-11-14T10:29:32.078+0900#011INFO#011[license]#011licenser/es_callback.go:50#011Elasticsearch license: Basic
2022-11-14T10:29:32.078+0900#011INFO#011[index-management]#011idxmgmt/manager.go:84#011Overwrite ILM setup is disabled.
```

—> elasticsearch.yml에 node.ingest가 false인지 체크한 후, true로 변경하고 모두 재시작하니 해결~

```yaml
node:
  master: true
  data: true
  ingest: true
```

## Error2

서비스 올리고 아래와 같은 에러가 찍히면서 서비스 연결이 안된다면,

```prolog
{"log.level":"error","@timestamp":"2023-11-10T08:50:53.903+0900","log.origin":{"file.name":"go-docappender@v0.2.1-0.20230829163624-c69a1cf8ce35/appender.go","file.line":278},"message":"bulk indexing request failed","service.name":"apm-server","error":{"message":"server shutting down"},"ecs.version":"1.6.0"}
```

-> 버전 호환문제로 apm을 7.6.2버전으로 다시 설치하니 해결됨

# reference

[Monitoring Applications with Elasticsearch and Elastic APM](https://www.elastic.co/kr/blog/monitoring-applications-with-elasticsearch-and-elastic-apm)