---
layout: post
title: WebSocket connection error - nginx
date: 2022-03-31 22:00:00
last_modified_at : 2022-03-31 22:00:00
parent: Home
# parent: Etc
has_children: false
nav_exclude: true
tags: [websocket, nginx]
---

### problem
nginx를 통해서 ui와 api간 ws통신하는데, 

개발자 도구에서 아래와 같은 에러메시지가 계속 발생하고 401 오류가 난다.

![ws1](../img/ws1.png)

![ws2](../img/ws2.png)

### error

```
WebSocket connection to 'ws:127.0.0.1:9091/websocket/112/gfwerwer/websocket?access_token=asdasdasdasda' failed:
```

### cause

웹소켓 통신을 하려면 헤더에 이 패킷이 upgrade될 패킷임을 웹서버가 알수 있도록 설정이 필요한대

그게 nginx.conf에 누락되어 있었음

그래서 바꾼 nginx.conf

```
 [nginx.conf]
server {
		listen 9091;

		location ~ /websocket/[0-9]+/ {
			proxy_pass http://localhost:9092;

			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "Upgrade";
		}

		location / {
			proxy_pass http://localhost:9092;
		}
	}
```

### reference

[nginx 에서 WebSocket Proxy 설정하기](https://shinwusub.tistory.com/m/111)