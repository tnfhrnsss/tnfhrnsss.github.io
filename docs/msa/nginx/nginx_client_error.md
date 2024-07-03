---
layout: post
title: 413 Request Entity Too Large & client intended to send too large body
description: client intended to send too large body
date: 2022-04-08 21:13:00
last_modified_at : 2022-04-08 21:13:00
parent: Nginx
grand_parent: Msa
has_children: false
nav_exclude: true
tags: [nginx, 413]
keywords: nginx
---

### problem

front에서 첨부파일을 끌어서 업로드할 때, nginx에서 에러 발생하면서 아무 반응 없던 현상

### nginx error

```
client intended to send too large body: 16095313 bytes
```

### server error

RestTemplate으로 파일 업로드를 하는 서비스에서 \
**413 Request Entity Too Large** 가 발생한다.

```
org.springframework.web.client.HttpClientErrorException: 413 Request Entity Too Large: 
"<html><EOL><EOL><head><title>413 Request Entity Too Large</title></head><EOL><EOL><body><EOL><EOL><center><h1>
413 Request Entity Too Large</h1></center><EOL><EOL><hr><center>nginx/1.25.4</center><EOL><EOL></body><EOL><EOL></html><EOL><EOL>"
at org.springframework.web.client.HttpClientErrorException.create(HttpClientErrorException.java:145)
at org.springframework.web.client.DefaultResponseErrorHandler.handleError(DefaultResponseErrorHandler.java:168)
at org.springframework.web.client.DefaultResponseErrorHandler.handleError(DefaultResponseErrorHandler.java:122)
at org.springframework.web.client.ResponseErrorHandler.handleError(ResponseErrorHandler.java:63)
at org.springframework.web.client.RestTemplate.handleResponse(RestTemplate.java:825)
at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:783)
at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:717)
at org.springframework.web.client.RestTemplate.postForEntity(RestTemplate.java:474)

```

![nginx2](../img/nginx2.png)

### solved

nginx.conf에 아래 설정 추가

```
http {
	
	client_max_body_size 500M;
}
```

0 으로 설정하면 무제한

500M으로 한 이유는 제품기획상 대용량 첨부가 불가능해서..

무제한으로 했다가 화면 뻗을 것 같아서 그러했음

### reference

[Nginx error: client intended to send too large body](https://stackoverflow.com/questions/44741514/nginx-error-client-intended-to-send-too-large-body)