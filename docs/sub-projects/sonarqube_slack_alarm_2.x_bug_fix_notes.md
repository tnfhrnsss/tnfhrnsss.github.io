---
layout: post
title: [sonarqube slack alarm] 2.x Snapshot bug fix notes
date: 2024-06-03 10:59:00
last_modified_at : 2024-06-03 10:59:00
parent: Sub Projects
has_children: false
nav_exclude: true
keywords: sonarqube
---

# (issut #3) additionally, matching Slack users with SonarQube using user Id.
- [#3](https://github.com/tnfhrnsss/sonarqube_slack_alarm/issues/3){:target="_blank"}
- 반영일자 : 2024-05-31
- 내용
    - 프로젝트가 github에 repo를 두고 있다면, codesmell로 검출된 라인의 author가 “`156968135+{id}@users.noreply.github.com`” 정보로 저장된다.
    - 하지만 맵핑할 slack userId는 `{id}@회사 도메인 주소`로 되어있다.
- 문제
    - 코드 author 정보와 slack userId가 mis match되면서 알림이 왔을때 제대로 작업자에게 멘션이 안가는 문제가 있다.

    ![2.x_bug_fix_notes_3.jpeg](../img/2.x_bug_fix_notes_3.jpeg)
    
- 조치 방안
    - 1안) 로컬 프로젝트에서 git config를 통해 이메일 주소를 설정하면 된다.
        - 문제) 글로벌이 아닐 경우 매번 설정필요
    - 2안) github 이메일 주소의 패턴을 통해서 slack user Id와 match시키는 방법 (택)
- 작업 내용
    - 2안의 내용으로 진행했다.
    - deep match서비스 생성
    - github 이메일 주소의 아이디에 slack user Id가 포함되는지 체크한다.