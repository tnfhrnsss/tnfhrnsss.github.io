---
layout: post
title: static하게 만들기 using nginx
description: static하게 만들기 using nginx
date: 2023-09-09 21:54:00
last_modified_at : 2023-09-09 21:54:00
parent: Nginx
grand_parent: Msa
has_children: false
nav_exclude: true
---

# 요구사항

- static한 이미지이지만 API를 통해서 매번 호출해야만하는 케이스에 대한 성능 개선
    - 정적인 파일로 되어있지만, 이미지를 클라이언트에 전달하려면 서버를 매번 호출해야하고
    - 해당 요청이 반복적으로 일어날것이 예상되는 상황
    - 이미지 크기가 큰 경우, 조회할 때마다 딜레이가 발생하는 문제

# 작업 내용

## nginx Reverse Proxy Cache 사용

- 기존 구조대로라면, 앱에서 spring boot 서버로 바로 이미지를 가져와야해서 네트워크 부담이 예상되는 구간이였습니다.
- 웹 서버를 통해서 요청이 가기전에 필요에 따라서 요청 내용을 변경하거나 캐싱을 해서 성능을 향상 시킬 수 있습니다
- nginx를 통해 응답을 캐싱해서 서버에 대한 부하를 줄이고 응답시간을 단축시킬 수 있습니다.
- proxy_cache, proxy_set_header 등과 같은 지시어를 사용하여 프록시 동작을 세밀하게 제어할 수 있습니다.

```yaml
location /static/depot/ {
  access_log  logs/faq.error.log main;
  proxy_set_header X-Authority "ADMIN";
  proxy_set_header X-User "system";
  proxy_pass http://x.x.x.x:8081$uri;
}
```

## Forward Proxy or Reverse Proxy

### Forward Proxy

- 클라이언트가 외부 서버 또는 리소스에 접근하기 위해 중간에 위치한 프록시 서버를 사용하는 것
- 클라이언트가 요청을 프록시 서버로 보내고, 프록시 서버는 클라이언트 대신 요청을 외부 서버로 전달하고 응답을 받아 클라이언트로 전달한다. 
- Forward proxy는 주로 클라이언트의 익명성을 보호하거나 내부 네트워크에서 외부 리소스에 접근할 때 사용
- 주로 resolver로 DNS 서버를 지정하게 된다.

### Reverse Proxy
- 웹 서버 뒷단에 위치하며, 클라이언트 요청을 받아 웹 서버로 전달하고, 웹 서버에서 받은 응답을 캐시에 저장하여 이후 요청에 사용하는 역할을 한다.
- 이는 웹 서버의 성능을 향상시키고 트래픽을 줄이는데 사용된다. Reverse proxy cache는 서버 측에서 동작하며 클라이언트 측에 대한 중개 역할을 하지 않는다.
-  Reverse proxy cache를 구현하려면 Nginx를 reverse proxy로 설정하고, 웹 서버의 응답을 캐시로 저장하도록 구성해야 한다.
- 밑에 설정 예시를 보면, 디스크에 저장한다는 것을 알수 있다.

  ```
    http {

    # 캐시 경로와 캐시 옵션을 설정
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m;

    server {
        ...

        location / {
            # 캐시 설정
            proxy_cache my_cache;
            proxy_cache_valid 200 304 1h;  # 200 OK 및 304 Not Modified 응답을 1시간 동안 유지
            proxy_cache_use_stale error timeout invalid_header updating http_500 http_502 http_503 http_504;
            
            proxy_pass http://backend_server;
        }
        
        location /clear_cache {
            # 캐시를 비우는 엔드포인트를 설정
            proxy_cache_purge my_cache "$uri$is_args$args";
        }

        location /status {
            # 캐시 상태를 모니터링할 수 있는 엔드포인트
            proxy_cache_bypass $http_cache_control;
            add_header X-Cache-Status $upstream_cache_status;
        }
    }

    upstream backend_server {
        server 127.0.0.1:8080;
    }
}

  ```

### 주의할 점

- 경우에 따라서는 expires 설정을 추가하는 것이 좋습니다.
- 404의 경우도 캐싱될 수 있기 때문에 주의!

# reference

[https://jojoldu.tistory.com/60](https://jojoldu.tistory.com/60)