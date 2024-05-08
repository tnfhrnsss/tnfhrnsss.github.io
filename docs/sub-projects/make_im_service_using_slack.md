---
layout: post
title: make im(identity management) service using slack api
description: make im(identity management) service using slack api
date: 2024-04-29 23:47:00
last_modified_at : 2024-05-08 10:54:00
parent: Sub Projects
has_children: false
nav_exclude: true
keywords: slack
---

# 배경

- 모델오피스 및 영업데모, 개발 서버의 workspace를 사용하기 위해 직원 계정을 수기로 json파일을 만들어서 관리하고 있다.
- 직원 정보가 변경(입사 or 퇴사)이 있을 때마다 수동으로 변경 내용을 작성해서 github에 푸시하고 init-data로써 적용시켜야하는 과정이 발생한다.

# 목표

- 슬랙에 조인한 멤버와 workspace 상담사 계정을 동기화하도록 한다.
- slack-im이라 하기엔 기능이 미미하지만 😭 계속 리팩토링하면서 키워봐야겠다!!
- 이번엔 스프링이 아닌 python으로 간단하게 작성해보자

# Repository

- 깃헙 [repository](https://github.com/tnfhrnsss/slack-im){:target="_blank"}에서 확인할 수 있다.

# 설계

## 1. 대상 서버

- 영업데모
    - 부정기적으로 설치되기 때문에 배치로 동기화하는 것으로 한다.
- 모델오피스
    - 2주마다 스프린트 종료할 때 jenkins를 통해 자동 배포되고 있다.
- 개발서버
    - 매일 12시에 jenkins에 의해 초기화 및 배포되고 있다.

## 2. 동기화 방법

- 간접 호출 : 각자의 jenkins job을 수행하고 post action으로 진행
    - 일배치로만 해도 될것같다. (신규/퇴사자가 바로 발생하지 않으므로)
- 직접 호출 : jenkins pipeline 으로 넣고 직접 호출하도록 한다.

## 3.  슬랙 User API

- bot oauth 로 users:read , users:read:email가 있어야한다.
- 전사 채널을 대상으로 한다.
- 필터 조건
    - 봇 계정인 경우
    - 삭제된 계정인 경우
    - 이메일 인증하지 않은 계정인 경우

# 환경

- python 3.x
- slack sdk
- jenkins 2.414.1

# 작업내용

- 모듈 설치

```java
pip install slack_sdk
pip install requests
```

- config 설정
    - 슬랙 토큰 정보
    - API 및 payload 정보
- slack user api 호출
    
    ```python
    def findAll():
        slack_token = config['slack_token']
        client = WebClient(token=slack_token)
    
        response = client.users_list()
        filtered_users = []
    
        if response["ok"]:
            users = response["members"]
            for user in users:
                deleted = user["deleted"]
                isBot = user["is_bot"]
                if deleted == True or isBot == True:
                    continue
    
                isEmailConfirmed = user["is_email_confirmed"]
                if isEmailConfirmed == False:
                    continue
    
                filtered_users.append(user)
            return filtered_users
        else:
            print("Failed to fetch users from Slack.")
    ```
    
- service api 호출
    - 아직은 고정 payload로만 작성되어있다. (리팩토링 예정)
    
    ```python
    def register(user_id, user_name):
        if len(user_id) == 0:
            print("user_id is null. {}", user_name)
    
        headers = {
            
        }
        payload = {"userId": user_id, "name": user_name, "password": "1", "roleGroupId": "ADMIN", "centerIds":["spectra"]}
        response = requests.post(api_url, headers=headers, json=payload)
    
        if response.ok:
            print("User {} registered successfully.".format(user_name))
        else:
            print("Failed to register user .{}.".format(user_name))
    ```
    

# 실행

## 명령어
    
    ```java
    python3 sync.py ${API_URL}
    ```
 
## add jenkins job using pipeline
- util-slack-im라는 job을 추가한다고 하면
- pipeline에 trigger되어서 파이썬 스크립트를 실행할 수 있게 한다.

  ```
    pipeline {
    stages {
      stage('INITIALIZE') {
        steps {
          echo "PATH = ${PATH}"
          echo "MAVEN_HOME = ${MAVEN_HOME}"
          echo "${params.API_URL}"
        }
      }
      stage('INIT DATA - SLACK IM') {
        steps {
          sshPublisher(
            publishers: [
              sshPublisherDesc(configName: 'test', verbose: true,
                transfers: [
                  sshTransfer(
                    execCommand:
                      """
                        cd /home/tools/util-cstalk-im/script
                        python3 sync.py ${params.API_URL}
                      """
                  )
                ]
              )
            ]
          )
        }
      }
    }
    options {
      timeout(time: 5, unit: 'MINUTES')
    }
  }
  ```

- upstream job에서 API_URL 파라미터를 포함해서 위에 추가한 job을 호출한다. 

  ```
  steps {
      build job: "util-slack-im", wait: false, parameters: [[$class: 'StringParameterValue', name: 'API_URL', value: 'http://localhost:8080/api']]
  }
  ```

# 남은 작업

- 배치시간을 시스템별로 다를 수 있다.
- api와 payload가 시스템별로 다를 수 있다.
- 동기화하기 전에 중복 체크한다.

# Error

## 1. NotOpenSSLWarning: urllib3 v2 only supports OpenSSL 1.1.1+, currently the 'ssl' module is compiled with 'LibreSSL 2.8.3’
    - [urllib3-v2-0-only-supports-openssl-1-1-1-에러-해결-c750d17bf3a6](https://medium.com/@tlsrid1119/urllib3-v2-0-only-supports-openssl-1-1-1-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0-c750d17bf3a6) 보고 urllib3 다운그레이드 함
    
    ```java
    pip uninstall urllib3 
    pip install 'urllib3<2.0'
    ```

## 2. how to pass the parameter from upstream job to downstream job
- upstream의 parameter를 downstream job으로 전송하는 것이 잘 안되었는데..
- https://stackoverflow.com/questions/63370946/jenkins-canot-cannot-pass-parameters-to-the-remote-servers-shell-script-with-p 에서 알수 있었다.
  - '''에서 """로 변경해야 한다. 


# Reference

- [https://api.slack.com/methods/users.list](https://api.slack.com/methods/users.list)
- [baeldung - jenkins-pipeline-trigger-new-job](https://www.baeldung.com/ops/jenkins-pipeline-trigger-new-job)