---
layout: post
title: importing evernote to notion
date: 2023-01-28 00:15:45
last_modified_at : 2023-01-28 00:15:45
parent: Etc
has_children: false
nav_exclude: true
---

## 작업 후기

- 에버노트를 2010년부터 사용했기 때문에 약 2천개의 노트가 있었는데요. 현재는 github으로 옮기면서 약 1100개가 남아있는 상태입니다.
- 노션은 2021년부터 사용하기 시작했는데요. 사용하다보니 노션만 쓰게되어서 이참에 에버노트를 정리하고 탈퇴하려고 합니다.
- 에버노트가 몇 년전부터 사용자경험도 안좋아지고 결제를 자꾸 요구하고, 업데이트하라는 알림도 너무 자주 보여서 점점 사용하기 불편해졌습니다.
- 게다가 에버노트를 실행하면 단축키가 다른 프로그램과 겹쳐서 이것도 저한텐 별로였습니다.

## how to import evernote

- 노션에서 가져오기 한 후 에버노트 계정 연결하면, “Evernote에서 1개의 노트북을 가져오는 중”에서 멈추는 현상이 발생합니다.

![importing_evernote.png](../img/importing_evernote.png)

- 문제는 에버노트의 노트 내용에 있었는데요. 아래처럼 노트를 수정하고 다시 [가져오기]하면 성공할 수 있습니다.
    - 에버노트의 노트에 첨부파일이 있다면 제거한다
    - 웹주소가 clip되어있다면 제거한다.
    - 내용에 테이블이 있다면 제거한다.
    - 내용에 “서식제거” 및 “서식 간단히”를 한다.

- 만약에 위의 방법으로도 해결되지 않는다면, 노트들을 html파일로 내보내기 한 후, 노션에서 html로 가져오기하면 됩니다.
    - 대신 이 방법은 노트 생성일자 등 노트의 정보는 가져올 수 없습니다.

## reference

[https://timelyphoto.com/blog/2021/how-to-fix-evernote-to-notion-import](https://timelyphoto.com/blog/2021/how-to-fix-evernote-to-notion-import)