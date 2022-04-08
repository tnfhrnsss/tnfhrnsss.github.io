---
layout: post
title: client intended to send too large body
date: 2022-04-08 21:13:00
last_modified_at : 2022-04-08 21:13:00
parent: Errors
has_children: false
nav_exclude: true
tags: [nginx]
---

### problem

front에서 첨부파일을 끌어서 업로드할 때, nginx에서 에러 발생하면서 아무 반응 없던 현상

### error

```
client intended to send too large body: 16095313 bytes
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