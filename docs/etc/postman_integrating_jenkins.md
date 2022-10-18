---
layout: post
title: postman api test and integrating with jenkins
date: 2022-10-18 22:11:00
last_modified_at : 2022-10-18 22:11:00
parent: Etc
has_children: false
nav_exclude: true
tags: [postman, jenkins]
---

# Goal

- layer가 다른 service끼리 dependency가 없는 상태에서 client 호출시 API 변경사항에 대한 대응개발을 감지하기 위한 목적으로 작성했습니다.
- 매 스프린트마다 수동으로 API테스트를 진행했었는데, 자동화를 하려고 합니다
- postman으로 작성한 후, jenkins의 job으로 등록해서 주기적으로 체크하려고 합니다.

# Step

## [postman]

1. collection를 생성하고 단위테스트가 필요한 API를 작성합니다.
2. 의존관계가 있는 API는 순서대로 작성해야 합니다. 
3. 작성이 완료되면 collection에서 export를 하고 파일을 다운받습니다. (url형식으로 실행시킬 수 있지만, 저는 내부망에서 실행해야하기 때문에 파일로 받았습니다.)

![pij1.png](../img/pij1.png)

## [Jenkins]

### 1. install nodejs, newman

1. jenkins를 설치했으면, jenkins에 nodejs를 설치합니다.
    1. Jenkins 관리 > Plugin Manager로 이동해서 nodejs로 검색하면 설치할 수 있는데요. 이 기능을 사용하면 latest로 설치되기 때문에 gcc오류가 날 수 있습니다.
    2. 그래서 저는 stable버전으로 설치하기 위해 여기서 hpi파일을 다운받습니다.
        
        ```yaml
        https://archives.jenkins-ci.org/plugins/nodejs/latest/
        ```
        
    3. [Jenkins 관리 > Plugin Manager > 고급 탭]에서 Deploy Plugin에 위에서 다운받은 파일을 불러옵니다.  
        
        ![pij2.png](../img/pij2.png)
        
2. nodejs설치가 되었으면, nodejs로 newman을 설치합니다.
    1. [Jenkins관리 > Global Tool Configuration]으로 이동하면 Nodejs가 있습니다.
    2. Add NodeJS 를 눌러서 상세 설정합니다.
    3. 저는 16.18.0 버전이 필요해서 버전을 16.18.0으로 하고 newman을 설치하는 것으로 했습니다
        
        ![pij3.png](../img/pij3.png)
        

### 2. create new item

1. Freestyle project로 item을 하나 생성합니다. (pipeline로 해도 됩니다)
2. 위에서 다운받은 파일을 생성한 프로젝트의 workspace하위에 위치시킵니다.
    1. 저는 test이름으로 생성해서 workspace가 이렇게 되었습니다.
        
        ```yaml
        /var/lib/jenkins/workspace/test
        ```
        
3. 빌드 환경에서 [**Provide Node & npm bin/ folder to PATH**]을 체크하고 위에서 설치한 nodejs를 선택합니다.
    
    ![pij4.png](../img/pij4.png)
    
4. Build Steps에서  [Execute shell]을 선택합니다.
    1. command에 newman으로 postman을 통해 작성한 스크립트를 실행시키도록 작성합니다
        
        ```yaml
        newman run crema-business.postman_collection.json --reporters cli,junit --reporter-junit-export report.xml
        ```
        

### 3. generate a test report file

1. xml로 report가 생성되게 하면 실패 성공 여부를 ui를 통해 확인할 수 있는데요. 아래처럼 에러가 날 경우
    
    ```yaml
    ERROR: Step ‘Publish JUnit test result report’ failed: No test report files were found. Configuration error?
    Finished: FAILURE
    ```
    
    1. command의 오타 확인하고 
    2. workspace의 위치를 다시 파악합니다. jenkins홈 위치가 아니라면, Configuration > General > Advance Option 에서 use custom workspace를 체크하고 위치를 지정해줍니다.
        1. 
        
        ![pij5.png](../img/pij5.png)
        
2. 그리고 Configuration > 빌드 후 조치에서 [**Publish JUnit test result report**]를 선택하고 위에서 command로 작성한 xml파일명을 입력합니다.
3. 빌드 실행 후 생성된 파일을 확인합니다. 

### view test result

- test result trend를 표시하게 하려면, 성공 결과가 있어야 합니다.

# Reference

[[DevOps] Jenkins/Postman/Newman으로 API 테스트 자동화하기](https://medium.com/dtevangelist/devops-jenkins-postman-newman%EC%9C%BC%EB%A1%9C-api-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%9E%90%EB%8F%99%ED%99%94%ED%95%98%EA%B8%B0-f08a155a949c)

[Run Newman in jenkins](https://stackoverflow.com/a/53142521/14257397)

[How to Generate Newman Reports on Jenkins?](https://www.toolsqa.com/postman/generate-newman-reports-on-jenkins/)

## error

```prolog

Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/test
Unpacking https://nodejs.org/dist/v18.11.0/node-v18.11.0-linux-x64.tar.gz to /var/lib/jenkins/tools/jenkins.plugins.nodejs.tools.NodeJSInstallation/nodejs_18.11.0 on Jenkins
$ /var/lib/jenkins/tools/jenkins.plugins.nodejs.tools.NodeJSInstallation/nodejs_18.11.0/bin/npm install -g newman
node: /lib64/libm.so.6: version `GLIBC_2.27' not found (required by node)
node: /lib64/libc.so.6: version `GLIBC_2.25' not found (required by node)
node: /lib64/libc.so.6: version `GLIBC_2.28' not found (required by node)
node: /lib64/libstdc++.so.6: version `CXXABI_1.3.9' not found (required by node)
node: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.20' not found (required by node)
node: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by node)
[test] $ /bin/sh -xe /tmp/jenkins17489636526245755837.sh
/tmp/jenkins17489636526245755837.sh: line 2: unexpected EOF while looking for matching `''
Build step 'Execute shell' marked build as failure
Finished: FAILURE
```

- 원인은 nodejs 18버전 때문인 것 같다. 가장 latest를 설치하지 말고, NodeJS 16 LTS Version을 설치하라고 함