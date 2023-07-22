---
layout: post
title: When SonarQube Detects Code Smells, Send Slack DM
description: When SonarQube Detects Code Smells, Send Slack DM
date: 2023-07-22 17:57:00
last_modified_at : 2023-07-22 17:57:00
parent: Sub Projects
has_children: false
nav_exclude: true
keywords: sonarqube
---

# Goals

- 항상 릴리스 때 몰아서 처리해야했던 코드스멜을 일배치 점검을 통해서 빠른 릴리스될 수 있도록
- 검증 컴포넌트가 많다보니, 매번 소나큐브 콘솔 통해서 확인해야하는 불편함 해소
- 본인이 작업한 코드는 본인이 처리하도록 하는 독려와 강요 사이..

# 작업 내용
- API들은 feign client를 통해서 개발했다.
- 일단 jar로 만들고 API로 호출하도록 했다.

# SonarQube API

- 릴리스 대상 컴포넌트들은 yml에 설정하고 검증 대상으로 했다.
- 사용한 이슈 검색 search api
    - /api/issues/search
    - code-smell search 파라미터
    
    ```bash
    curl --location 'http://localhost:9100/api/issues/search?componentKeys=&s=FILE_LINE&resolved=false&types=CODE_SMELL&ps=100&facets=severities%2Ctypes&additionalFields=_all&timeZone=Asia%2FSeoul'
    ```
    
    - response
        
        ```bash
        {
            "total": 8,
            "p": 1,
            "ps": 100,
            "paging": {
                "pageIndex": 1,
                "pageSize": 100,
                "total": 8
            },
            "effortTotal": 40,
            "issues": [
                {
                    "key": "AYhwJmBB5K8chirpKye_",
                    "rule": "java:S4968",
                    "severity": "MINOR",
                    "component": "",
                    "project": "",
                    "line": 20,
                    "hash": "baa052d59356658032eda75e980c8241",
                    "textRange": {
                        "startLine": 20,
                        "endLine": 20,
                        "startOffset": 50,
                        "endOffset": 64
                    },
                    "flows": [],
                    "status": "OPEN",
                    "message": "Replace this type parametrization by the 'final' type `Long`.",
                    "effort": "5min",
                    "debt": "5min",
                    "author": "",
                    "tags": [],
                    "transitions": [],
                    "actions": [],
                    "comments": [],
                    "creationDate": "2020-09-28T14:55:09+0900",
                    "updateDate": "2023-05-31T13:51:24+0900",
                    "type": "CODE_SMELL",
                    "scope": "MAIN",
                    "quickFixAvailable": false,
                    "messageFormattings": []
                }
            ],
            "components": [
            ],
            "rules": [
            ],
            "users": [],
            "languages": [
            ],
            "facets": [
            ]
        }
        ```
        

# Slack API

## 1. Channel User List API

- [https://api.slack.com/methods/conversations.members](https://api.slack.com/methods/conversations.members)

```bash
curl --location 'https://slack.com/api/conversations.members?channel=xxxxx' \
--header 'Authorization: Bearer xxxxx'
```

## 2. User Profile API

- 권한이 user.read, user.profile.read, user.read.email이 있어야한다.
- [https://api.slack.com/methods/users.profile.get](https://api.slack.com/methods/users.profile.get)

```bash
curl --location 'https://slack.com/api/users.profile.get?user=xxxxx' \
--header 'Authorization: Bearer xxxxxxxx'
```

## 3. Send DM API

- 권한은 im이 있어야한다.
- 유저는 @ + id, 또는 2번 통해서 얻어낸 id를 사용
- [https://api.slack.com/methods/chat.postMessage#dm](https://api.slack.com/methods/chat.postMessage#dm)

```bash
curl --location 'https://slack.com/api/chat.postMessage' \
--header 'Content-Type: application/json; charset=utf-8' \
--header 'Authorization: Bearer xxxxx' \
--data-raw '{
    "channel":"@id",
    "text":"Hello, this is a direct message!"
}'
```


# Repository
https://github.com/tnfhrnsss/sonarqube_monitoring.git

# Output

![sonarqube_codesmell_slack_notification.png](./img/sonarqube_codesmell_slack_notification.png)

# 남은 할일

- 젠킨스 파이프라인으로 등록
    - 토큰 보안처리
- 코드 스멜이 많을 경우 페이징 처리 필요하다.
- 스프링 배치 추가
- python으로도 구현해야겠다!