---
layout: post
title: npm install failed
description: npm install failed
date: 2023-07-11 09:48:00
last_modified_at : 2023-07-11 09:48:00
parent: Errors
has_children: false
nav_exclude: true
keywords: npm
---

# 종속성 문제일 때

- 이 오류는 npm이 종속성 트리를 해결하지 못하는 경우에 발생
- 조치 순서
    - **`npm cache clean`** 명령을 실행하여 npm 캐시를 지우고 다시 시도
    - **`package-lock.json`** 파일을 삭제하고 **`npm install`**을 다시 실행. 이렇게 하면 종속성 트리를 다시 생성하게 된다.
    - 종속성 충돌을 해결하기 위해 **`npm ls`** 명령을 사용하여 현재 설치된 패키지의 종속성 트리를 검사한다. 종속성 간의 충돌이 있는 경우, 충돌을 해결하기 위해 패키지 버전을 업데이트하거나 **`npm update`**를 사용하여 종속성을 업데이트할 수 있다.
    - 패키지가 손상되었을 가능성이 있는 경우, 해당 패키지를 삭제하고 다시 설치한다. 이를 위해 **`node_modules`** 폴더를 삭제하고 **`npm install`**을 실행하여 패키지를 다시 설치
    - 프로젝트의 종속성 간의 충돌을 확인하고 해결할 수 있는 도구인 **`npm-check`** 또는 **`yarn`**을 사용해 본다.