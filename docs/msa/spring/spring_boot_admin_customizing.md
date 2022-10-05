---
layout: post
title: spring boot admin customizing
date: 2022-10-05 20:51:00
last_modified_at : 2022-10-05 20:51:00
parent: Spring
grand_parent: Msa
nav_exclude: true
tags: [spring, admin]
---

# How to customizing spring boot admin

## to do list

- 간단하게 spring admin에 커스텀 메뉴를 추가해봅니다.
- helpdoc : [spring-boot-admin customizing-custom-views](https://codecentric.github.io/spring-boot-admin/current/#customizing-custom-views)

ℹ️ 주의 : ui는 vuejs로만 가능합니다!

## setting

1. github에 가서 소스를 내려받아서 프로젝트 셋팅을 합니다. 

[spring-boot-admin](https://github.com/codecentric/spring-boot-admin/tree/master/spring-boot-admin-samples/spring-boot-admin-sample-custom-ui)

1. 저는 spring boot 2.7.4를 기준으로 작업했습니다.
2. eureka에 등록되어 작업하는 것을 기준으로 했을 경우에만 아래 내용을 적용하면 될 것 같습니다. 그 외의 방식은 별도의 doc문서를 참고하면 됩니다.

## add custom code

1. ${project_HOME}\spring-boot-admin-samples\spring-boot-admin-sample-custom-ui\src\index.js 수정
    1. `scheduler`란 메뉴를 추가한다고 하면, index.js파일에 SBA.use를 통해서 아래처럼 추가합니다.
        
        ```jsx
        import scheduler from './scheduler/viewScheduler';
        
        SBA.use({
          install({viewRegistry}) {
            viewRegistry.addView({
              name: 'scheduler',
              parent: 'instances', // <1>
              path: 'scheduler',
              component: scheduler,
              label: 'Scheduler',
              order: 1000,
              //isEnabled: ({instance}) => instance.hasEndpoint('custom') // <2>
            });
          }
        });
        ```
        

1. spring-boot-admin-sample-custom-ui 프로젝트의 src폴더 하위에 커스텀 페이지를 작성합니다. 
    1. 저는 quartz구현된 scheduler서비스에 등록된 job리스트를 보여주기 위한 페이지를 생성했습니다.

## build custom-ui project

1. 작업이 다 되면, spring-boot-admin-sample-custom-ui 폴더 하위에서 아래 명령어를 차례대로 실행합니다.
    
    ```bash
    $ yarn install
    
    $ npm run watch
    ```
    
2. 다 되면 target폴더 하위에 dist 폴더가 생성되면서 spring-boot-admin-sample-custom-ui-2.1.1-SNAPSHOT.jar가 생성됩니다.

## load custom-ui

1. 기존 spring-boot-admin 서비스에 custom작업했던 페이지를 add하는 방법은 3가지 정도 파악했는데요.
    1. 방법1) yml에 아래 속성을 추가합니다.
        
        ```yaml
        spring.boot.admin.ui.cache.no-cache: true
        spring.boot.admin.ui.extension-resource-locations: file:D:/_source/spring-boot-admin-2.1.1/spring-boot-admin-samples/spring-boot-admin-sample-custom-ui/target/dist/
        spring.boot.admin.ui.cache-templates: false
        ```
        
    2. 방법2) 위에서 생성된 jar를 외부 jar로 dependency추가하는 방법이 있습니다.
    3. 방법3) spring-boot-admin-server-ui를 npm run build해서 생성된 spring-boot-admin-server-ui-2.1.1-SNAPSHOT.jar로 서비스를 대체해서 띄우는 방법이 있습니다.

## done!

![sbac](../img/sbac.png)

scheduler 명으로 메뉴가 추가된 모습입니다.

데이터도 fetch를 통해서 db에서 가져오게 했습니다.

(ui 헤더가 다 interval인건…ㅋㅋ)

ui작업은 몇 줄안되기도 해서 금방했는데요

vue-cli-service오류가 계속 나서, 해결하는데 시간이 좀 걸렸습니다..

제품 구축에 도움이 될만한 정보를 추가하는 것으로 작업해봐야겠습니다. 

# Etc,.

## error

1. 스프링 버전업 하고 나서 콘솔에 이런 오류 발생. 내 로컬만 발생
    
    ```yaml
    TypeError: this.endpoints.findIndex is not a function
        at t.value (instance.js:48:27)
        at applications-list-item.vue:246:1
        at Array.some (<anonymous>)
        at o.hasShutdownEndpoint (applications-list-item.vue:246:36)
        at o.A (applications-list-item.vue:1:3288)
        at t._render (vue.runtime.esm.js:2654:28)
        at o.r (vue.runtime.esm.js:3844:27)
        at t.get (vue.runtime.esm.js:3415:33)
        at t.run (vue.runtime.esm.js:3491:30)
        at Kr (vue.runtime.esm.js:4090:17)
    ```
    
    - 원인파악
        - Journal 메뉴를 눌러서 Event중 ENDPOINTS_DETECTED를 보니까
        
        ```yaml
        {
            "endpoints": {
                "endpoints": {
                    "integrationgraph": {
                        "id": "integrationgraph",
                        "url": "http://172.16.100.130:8889/actuator/integrationgraph"
                    },
        ```
        
        - 이렇게 endpoints 하위에 또 endpoints가 있어서 parsing오류가 난 것으로 보입니다.
        - 아래 설정의 순서에 의해서 endpoints가 중복으로 생성되고 있었습니다.
            
            ```yaml
            spring.cloud.config.server.native.search-locations:
            	- classpath:/configuration/
            	- file:///D:/_GIT/branch/develop/config/core-asset-resource/registry-config/
              - file:///D:/_GIT/branch/develop/config/talk-resource/registry-config/
            ```
            
            - 에러가 났을 때는 classpath:/configuration/가 맨 하위에 있었는데요. 중복으로 설정이 로딩되면서 endpoint가 연속적으로 들어간 것으로 의심됩니다.

# reference

[ZUM-Pilot-advanced_quartz_scheduler_admin/](https://zuminternet.github.io/ZUM-Pilot-advanced_quartz_scheduler_admin/)

[Spring Boot Admin Reference Guide](https://codecentric.github.io/spring-boot-admin/current/#customizing-custom-views)

[exploit-spring-boot-admin-sba-good-way-preeti-gupta](https://www.linkedin.com/pulse/exploit-spring-boot-admin-sba-good-way-preeti-gupta)