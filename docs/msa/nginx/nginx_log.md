---
layout: post
title: Change Nginx Log Level
date: 2022-07-07 20:48:00
last_modified_at : 2022-07-07 20:48:00
parent: Nginx
grand_parent: Msa
has_children: false
nav_exclude: true
tags: [nginx log]
---

# nginx.conf로 로그레벨 및 로그 정보 수정

## log level 수정

[nginx conf log format](https://searchtool.tistory.com/16)

```
[nginx.conf] 
# 주석해제하고 debug로 변경
error_log  logs/error.log  debug;
```

- debug로 변경하면 아래처럼 상세하게 남겨진다

![nginx_log](../img/nginx_log.png)

## log format 추가

```bash
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    client_max_body_size 500M;
    server_tokens off;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"'
                      '"$request_uri" "$arg_url"'
                      '"$http_url"'
                      '"$request_length" "$bytes_sent"';

}
```

- $request_uri : full original request URI (with arguments)
- $arg_url : 파라미터 url이 존재할 경우 파라미터값 로깅 (ex. arg_파라미터명)
- $http_url : 헤더에 url이란 key로 추가한 경우 로깅 (ex. http_헤더)
- $request_length : 요청 길이 (요청 라인, 헤더 및 요청 본문 포함)
- $bytes_sent : 클라이언트에 전송 된 바이트 수