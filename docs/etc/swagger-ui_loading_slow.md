---
layout: post
title: swagger-ui loading slow
date: 2023-05-11 19:52:00
last_modified_at : 2023-05-11 19:52:00
parent: Etc
has_children: false
nav_exclude: true
---

# problem

- swagger-ui를 통해서 api테스트를 하려고 하는데, api가 적은수임에도 화면 로딩이나 execute자체가 느린 현상이 발생했다.

# try1. use custom ui

- 커스텀 UI 사용하는 방법으로  ReDoc이나 Spectacle 같은 UI를 통해 커스텀 할 수 있다고 한다.

# try2. delete un-used tags

- 불필요한 태그들은 삭제한다.
    - 기본 화면에는 end point, sample, parameter 등등을 표시하기 때문에 API 수에 따라 페이지가 엄청 길어질 수 있다. 사용안하는 태그들은 정리하면 속도에 도움될 수 있다.

# try3. use ‘deepLinking’ option

- 화면 지연로딩 옵션을 설정할 수 있다. 그래서 필요한 부분만 먼저 로딩하고 나머지는 호출될 때 로딩하도록 하는 방법
- 적용방법
    - properties나 yml에 적용한다.

```yaml
springdoc.swagger-ui.deepLinking=true
```

- 단점
    - deep linking 옵션을 사용하면, URL 주소창에서 직접 API 문서 페이지의 경로를 입력해서 접근할 수 있게 되고, 결국 이는 보안에 취약할 수 있다.