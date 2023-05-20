---
layout: post
title: Try nginx proxy_pass without dns server
date: 2023-05-20 12:32:00
last_modified_at : 2023-05-20 12:32:00
parent: Nginx
grand_parent: Msa
has_children: false
nav_exclude: true
tags: [nginx]
---

# problem

- dns 서버가 없는 환경에서 nginx를 통해 proxy_pass를 외부 도메인주소로호출해야해서 여러가지 방법을 고민해봤습니다.
- ip로 api를 호출하면 403 forbidden으로 리턴받는 상황이였습니다.
- 참고로 nginx는 일반적으로 **`/etc/hosts`** 파일을 직접 참조하지 않습니다. 따라서 dns 서버 없이 외부 도메인 주소를 호출하는 것은 nginx의 기본 기능으로는 지원되지 않습니다.

## try1. upstream 활용

```java
upstream chat-api {
    server aaa.com:443;
}

server {
		location ~* ^/(chat|talk) {
         proxy_pass https://chat-api$uri;
    }
}
```

—> 403 발생 ip로 443을 호출해서 403으로 리턴된 것 같다.

## try2. map 활용

```java
map $request_uri $new_uri
{
   /chat/write  https://aaa.com;
}
```

## try3. dnsmasq 활용

[https://rougeref.hatenablog.com/entry/2022/06/13/185501](https://rougeref.hatenablog.com/entry/2022/06/13/185501)

[https://qiita.com/yKanazawa/items/2136c4880e8d48370981](https://qiita.com/yKanazawa/items/2136c4880e8d48370981)

단점은 동적으로 hosts파일을 못읽어온다고 한다. 서비스 재시작해야 적용됨

## try4. unbound 활용

- 해보진 않았지만 dnsmasq와 비슷한 dns resolver 툴인 것 같습니다.
- 무료 오픈소스입니다.
- 설치하고 nginx에서 해당 DNS Resolver를 참조하도록 설정하면 도메인 이름을 IP 주소로 해석할 수 있게 됩니다.

# conclusion

- 위의 방법들로 여러 시도를 해본 결과 어떻게든 외부 dns로 방화벽을 뚫어서(예를 들면, kt나 google dns) 통신하거나
- 내부 dns 서버를 구성하거나 해야할 것 같습니다.
- 기타 검토한 것들입니다.
    - [https://stackoverflow.com/q/62703337/14257397](https://stackoverflow.com/q/62703337/14257397) → 안됨
    - [nginx-proxy-pass-의-aws-elb-연결-설정-f0c4b792ef71](https://circlee7.medium.com/nginx-proxy-pass-%EC%9D%98-aws-elb-%EC%97%B0%EA%B2%B0-%EC%84%A4%EC%A0%95-f0c4b792ef71) → 안됨