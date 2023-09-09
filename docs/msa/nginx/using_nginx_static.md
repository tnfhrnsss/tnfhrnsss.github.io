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

## nginx forward proxy 사용

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

### 주의할 점

- 경우에 따라서는 expires 설정을 추가하는 것이 좋습니다.
- 404의 경우도 캐싱될 수 있기 때문에 주의!

# reference

[https://jojoldu.tistory.com/60](https://jojoldu.tistory.com/60)