---
layout: post
title: [solved] nginx 65: No route to host
description: nginx 65 No route to host
date: 2025-07-07 21:51:00
last_modified_at : 2025-07-07 21:51:00
parent: Nginx
grand_parent: Msa
has_children: false
nav_exclude: true
tags: [nginx, 502, 65]
keywords: nginx
---

# Problem

- 정말이지 원래는 잘되던 통신이였다. 😭
- 단순히 proxy_pass만 하는 nginx인데.
- 외부 API에서 내 로컬 nginx로 호출을 하면 요청 그대로 다른 PC로 포워딩하면 되는 부분이였는데.
- 언제부터인가
    - error.log에는 (65: No route to host) 오류가 발생했고,
    
    ```java
    [error] 68467#0: *1 connect() to 10.10.x.x:8060 failed (65: No route to host) 
    while connecting to upstream, client: x.x.x.x, 
    server: localhost, request: "POST /thirdparty/message HTTP/1.1", 
    upstream: "http://10.10.x.x:8060/thirdparty/message", 
    host: "crema.xxx.co.kr:19010"
    ```
    
    - access.log에는 http code가 502 bad gateway가 발생했다.
    
    ```prolog
    .. "POST /thirdparty/message HTTP/1.1" 502 157 "-" "Apache-HttpClient/4.5.2 (Java/1.8.0_312)" "-"
    ```
    

# Env

- nginx가 설치된 PC와 포워딩 받는 PC모두 Mac에 OS도 세콰이어다.
- curl로 호출해도 200이고
- nc -z로 호출해도 succeeded고
- ping도 ok
- netstat -an | grep 8050으로 포트 리슨인가 체크하고
- 방화벽도 모두 off인 상태
- 브라우저에서 호출해도 200
- nginx 로그를 debug로 올려도, 참고할만한 정보는 없었다.
- nginx는 homebrew로 설치해서 사용중이다.

# Solved

- perplexity에 질의를 했고, 여러가지 점검 포인트를 알려줬다.
- 권한 문제였다.
    
    ```prolog
    sudo nginx -s stop
    sudo nginx
    ```
    

# Cause

- sudo를 통해 서비스를 기동하니 통신이 되기 시작했다.
- 왜그럴까? brew로 실행했을때와 어떤 차이가 있길래, brew services restart nginx로는 통신이 막히고, sudo로는 되는걸까?
- perplexity의 답변
    - 실행 계정(사용자)의 차이
        - *brew services*로 실행하면, nginx가 일반적으로 **현재 로그인한 사용자 계정**(혹은 launchctl에 등록된 사용자)으로 실행됩니다.
        - *sudo nginx*로 직접 실행하면, nginx가 **root 권한**으로 시작되고, 이후 설정에 따라 **`user`** 디렉티브에 지정된 계정(예: nobody, nginx 등)으로 worker 프로세스가 실행됩니다.
- nginx외에도 redis, mysql 등에서 이런 케이스가 발생한다고 한다. 이럴때 sudo로 실행하면 해결된다고 하니 기억해야겠다.  👍