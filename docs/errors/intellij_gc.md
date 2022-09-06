---
layout: post
title: intellij gc overhead limit exceeded maven
date: 2022-09-06 22:13:00
last_modified_at : 2022-09-06 22:13:00
parent: Errors
has_children: false
nav_exclude: true
tags: [intellij]
---

![intellij_gc](../img/intellij_gc.png)

## problem

- 이유는 여러가지인 것 같은데, 이번에 발생한 이유는 종합적이였다. 주로 pull 받고나서 발생한다.
    - pom.xml의 version이 잘못되어서 전체 build 실패가 발생하거나
    - 패키지 경로가 변경된 파일을 pull받았을 때
- 원인을 그냥 수정했으면 해결되었을 것 같다. 근데 이것저것 clear하고 하다보니 점점 더 꼬이게 된 것 같다.
- 그래서 rebuild 및 build를 돌리면 이미 컴파일이 되어있음에도 intellij는 package not found에러를 전체 모듈개수만큼 발생시킨다. 전체적으로 고장난 느낌 ㅜㅠ

![intellij_gc1](../img/intellij_gc1.png)

## try 1

- 이런 유형의 직접적인 원인은 gc가 아닌 경우가 많았다.
- compiler 버전이 맞지 않았다거나
    - intellij > settings > build, execution, deployment → Compiler → Java Compiler
    
    ![intellij_gc2](../img/intellij_gc2.png)

## try 2

- 최근에 push된 git history를 보는 것도 좋다.
    - 아까처럼 pom.xml 오류가 발생했다거나
    - 새로운 module이 추가되었거나
    - 기존 module이 삭제되었는데 dependency 오류가 있다거나
    - mudule name이 변경되었다거나

## try 3. do not use “clear output directory on rebuild” option

![intellij_gc2](../img/intellij_gc2.png)

- 이번엔 이 옵션 해제로 해결! 어차피 수동으로 빌드할 때마다 clear하고 하기 때문에 해제해도 되었다.
- 추가로 “Rebuild module on dependency change” 옵션도 해제했다. pc 성능이 안좋다보니 자동으로 되는 것들이 오히려 문제 일으키는 것 같아서..

## try 3

다 안된다고 하면, 다른 경로에 파일을 새로 받아서 신규 프로젝트로 다시 생성하는 것도 방법이다. 
