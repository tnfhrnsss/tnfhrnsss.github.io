---
layout: post
title: 우아한테크세미나 - 개발자가 꼭 알아야 할 애플리케이션 보안
date: 2022-10-17 23:03:00
last_modified_at : 2022-10-17 23:03:00
parent: Youtube
grand_parent: Mooc
nav_exclude: true
---

## 강의 주소

[https://www.youtube.com/watch?v=RQv86D0M5YY](https://www.youtube.com/watch?v=RQv86D0M5YY)

## 추천 사이트

- 포스트위거(https://portswigger.net/web-security)
- 웹패킹.kr(https://webhacking.kr)
- 드림핵(https://dreamhack.io)

## 최근 취약점

1. IDOR(insecure direct object reference) 취약점
    1. 안전하지 않은 직접 객체 참조 내지 부적절한 인가
    2. 서버에서 수행하는 권한 검증 및 접근제어를 클라이언트에서 수행시 발생하는 문제
    3. 타 사용자의 객체에 접근해서 정보 유출 및 기능 조작 하는 케이스
    4. 조치방안
        1. 검증은 반드시 session 또는 서버에서 발급한 token을 통해 서버에서 수행하도록 한다
2. SSRF(server-side request forgery) 취약점
    1. 서버단에서 요청을 발생시켜 내부시스템에 접근하는 취약점
    2. 예를 들어 uri를 서버에서 직접 요청해서 외부 이미지를 가져오는 형태의 기능에서 발생
    3. public cloud환경에서는 가상서버의 메타데이터를 확인할 수 있는 API가 있어서 SSRF를 통해 해당 API를 요청해서 가상서버의 정보를 확인하는 방식으로 보완
    4. 조치방안
        1. 접근허용 ip를 whitelist로 필터링
        2. 반드시 필요한 uri scheme만 사용하도록 whitelist방식으로 필터링 필요
        3. loopback uri가 들어온 경우 필터링 필요
        4. private ip, link-local address형태의 uri가 들어온 경우 필터링 필요
        5. 불필요한 특수문자 @, %0a등이 uri에 있는 경우 필터링 필요
3. JWT인증 취약점
    1. 토큰이 탈취되었을 경우 임의로 서버에서 토큰을 만료시킬 수  없음
    2. 토큰을 localstorage에 저장해야할때 xss로 인한 탈취 가능성이 높음
4. Actuator Misconfiguration
    1. 불필요한 endpoint를 활성화시켜서, 예상치 못한 actuator의 기능으로 서비스 기밀성, 무결성, 가용성을 침해당할 수 있다.
        1. shutdown을 제외하고 모두 enable상태이기 때문에 필요한 것만 enable로 합니다.
    2. jmx는 모두 expose되어있고, http는 health만 expose되어있다.
    3. acuator에 접근할 수 있는 ip주소를 제한한다
    4. actuator 기본 경로를 변경한다.
    5. actuator 접근 시 권한 검증을 수행한다.
    6. 최종 보안 설정
        
        ```yaml
        management.endpoints.enabled-by-default = false
        management.endpoint.info.enabled = true
        management.endpoint.health.enabled = true
        management.endpoints.jmx.exposure.exclude = *
        management.endpoints.web.exposure.include = info, health
        management.server.port = (포트번호)
        management.server.address = (IP 주소)
        management.endpoints.web.base-path = (변경된 경로)
        ```
        

## 기타 질답

1. 대규모 프로젝트에선 was session보다 redis에 세션을 저장하고 각각 was에서 가져갈 수 있도록 한다.