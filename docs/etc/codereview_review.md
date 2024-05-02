---
layout: post
title: Code Review's Review
description: Code Review's Review
date: 2022-10-14 02:00:00
last_modified_at : 2024-05-03 02:00:00
parent: Etc
has_children: false
nav_exclude: true
---

**현재는 gitlab→github으로 이전하면서, pr을 통해 코드리뷰 하고 있다.**

# 검토기간 (22.10.14)

# Background

- 팀 내에서 업소스로 코드리뷰를 진행하고 있었는데, 리뷰활동이 저조해서 자동으로 검출할 수 있는 부분을 좀 더 늘리고자 고민해봤다.
- 코드 리뷰 활동이 저조했던 이유
    - 리뷰 문화가 정착되기 힘든 구조
        - 빡빡한 스프린트
        - 신속한 납품 지원
        - 구성원 간 리뷰활동 필요성 대한 이해 차이
    - 업소스 관리 어려움
        - 업소스 서비스 불안정성
        - 관리 리소스가 요구됨
- gitlab를 통한 코드리뷰 검토
    - intellij를 통해 mr하려면 버전을 올려야한다. (현재 버전 GitLab Community Edition 12.5.3)
    - 설치형이다보니 버전업이 쉽지않다.

# Try 1. 코드리뷰 활동 알림 서비스 강화

- 업소스의 webhook을 통해 이벤트가 발생하면 슬랙으로 알림 메시지를 전송한다.
- 코드리뷰 리포트 용도로도 활용할 수 있다.
    - 잔여 리뷰건
    - 기한 임박 리뷰건
    - 리뷰 요청/완료 알림
    - 팀별 리뷰 활동 인사이트 제공
- 적용
    - webhook 설정방법
        1. admin으로 로그인 후, 웹훅할 프로젝트 edit project 선택 → General의 오른쪽 webhooks 선택 후 이벤트 및 webhook url설정
        2. 웹훅 수신 프로젝트 생성

# Try 2. 코드리뷰 규칙 정하기

- 규칙을 정해서 문화정착에 도움이 되게 해보자
- 설계 리뷰일 경우 오프라인 권장
- 리뷰어는 1명~2명 권장
- 리뷰양은 적게
- 요청할 때 검토받고 싶은 부분을 잘 적기

# Try 3. 다른 코드리뷰 프로그램 검토

- 코드 스트림: [https://newrelic.com/kr/codestream](https://newrelic.com/kr/codestream)
- jetbrain space

# Try 4. 타사 코드리뷰 문화 조사

## 1. 구글

- 코드리뷰 자동화
    - Tricorder라는 정적 분석(잠재결함 분석) 도구와 Rosie라는 코드 정리 시스템(Large-Scale Cleanups and Code Changes)을 활용해서 사용하지 않는 코드 삭제
- 수동
    - 리팩토링(Refactoring)이 필요한 경우에는 수정된 코드를 개발자들에게 리뷰 요청
- Critique이라는 툴 사용
- **Code Reviews at Google are lightweight and fast**([code-reviews-at-google/](https://www.michaelagreiler.com/code-reviews-at-google/))
- 코드리뷰 가이드 : [https://google.github.io/eng-practices/review/](https://google.github.io/eng-practices/review/)

## 2. 마이크로소프트

- 자동화
    - Codeflow를 통해서 코드 리뷰 진행
    - [Codeflow](https://www.getcodeflow.com/#)에 가입후 gitlab을 연동해야하는 서비스형(설치형 아님)
- 수동 툴
    - VSCode의 여러 코드 리뷰 플러그인 활용
    - GitHub, Visual Studio Teams Service를 활용하는 등 다양한 방법으로 이용

## 3. 컬리

- 코드리뷰 정책([https://helloworld.kurly.com/blog/squad-b-team-building/#코드-리뷰](https://helloworld.kurly.com/blog/squad-b-team-building/#%EC%BD%94%EB%93%9C-%EB%A6%AC%EB%B7%B0))

## **4. 네카라쿠배*와 스타트업**

- 대부분의 회사는 GitHub의 PR을 활용해서 리뷰하는 방식으로 코드 리뷰를 진행
- 카카오
    - 자체 개발한 TestBot을 통해 Code Smell과 Bug, Vulnerability 등의 문제를 분석하여 알림으로 제공
    - Chrome Extension을 개발해 코드 리뷰가 필요할 때마다 코드 리뷰어를 자동 할당하여 리뷰하도록 하고 있음
- [https://devs0n.tistory.com/22](https://devs0n.tistory.com/22)

# 결론

- 수동 리뷰 외에도 자동화 부분을 늘리는 방향도 검토가 필요하다.
- archunit을 통해 의존성 관계나 네이밍 규칙을 검사해도 좋을 것 같다.