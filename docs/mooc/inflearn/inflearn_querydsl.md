---
layout: post
title: 실전 Querydsl 수강 정리
date: 2022-10-01 22:11:00
last_modified_at : 2022-10-01 22:11:00
parent: Inflearn
grand_parent: Mooc
nav_exclude: true
tags: [querydsl, jpa]
---

강사: 김영한

# 소개

- 문법 오류를 컴파일 시점에 잡아준다
- 동적 쿼리 문제 해결해준다
- 쉬운 sql스타일 문법

# 환경 설정

- 프로젝트 생성
    - dependency: web, jpa, h2, lombok
- Querydsl 설정과 검증
    - querydsl-jpa 추가
    
    ```yaml
    implementation 'com.querydsl:querydsl-jpa'
    ```
    
    - Q파일 생성되도록 테스트 코드 작성
        - Q파일은 git에 올리면 안된다.(빌드 폴더자체를 ignore시키면 됨)
- h2 설치

# 예제 도메인 모델