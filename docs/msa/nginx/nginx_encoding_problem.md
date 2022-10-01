---
layout: post
title: nginx query string encoding problem
date: 2022-07-23 23:01:00
last_modified_at : 2022-07-23 23:01:00
parent: Nginx
grand_parent: Msa
has_children: false
nav_exclude: true
tags: [nginx, encoding, feign]
---

## 요구사항

- ap서버에서 nginx를 통해서 외부 nas에 저장된 첨부파일을 가져와야하는 요건

```prolog
http://172.16.120.206:8051/?url=http%3A%2F%2Ftest.com%2Ftalkm%2FoY7TLhEsug%2FjqKpQBa0HtbvGdRjirexV0%2Fi_504b25b09b19.jpg
```

- nginx 서버 : 172.16.120.206:8051
- 파라미터 url을 AP서버에서 feignclient로 nginx 통해 호출해서 첨부파일을 읽어봐야 함
- feign으로 호출하기 전에는 인코딩이 되어있지 않지만, feign라이브러리 내 httpclient를 구현하는 클래스에서 인코딩이 시킨후 호출되고 있었습니다.

이런 형태의 url일 경우, regex로 matching시키는게 간단하지 않습니다.

dns가 변경될 가능성도 크고

보안 url이기 때문에 리소스 path가 유동적일 가능성도 큽니다.

## try1..

nginx에서 $args_url을 받자마자 proxy_pass하는 방법으로 젤 먼저 해봤습니다

### try1-problem

url에 인코딩된 특수문자로 에러가 발생합니다.

---

## try2..

슬래시가 한번일 경우 한번만 matching시키고 나머지는 (.*)$로 잡으면되는데

```
location ~* ^.+\.(jpg|jpeg|gif|png|pdf|txt|bmp)$ { 
```

이렇게 작성하면 조건을 타지도 못한다. 
이유는 query_string의 내용으로 location을 matching시킬수 없다고 한다.

## try3..

경로가 여러개 일 경우는 억지로 한다면 이렇게 하게 됩니다.

```bash
if ($args ~ (test.com)%2F(.*)%2F(.*)%2F(.*)%2F(.*)$) {
	  set $aaa $1;
    set $bbb $2/$3/$4/$5;

    rewrite . /$bbb break;
    proxy_pass http://$aaa;
}
```

### try3-problem

앞서 설명했듯이 url자체가 유동적일 가능성이 크기 때문에 위의 방법은 사용하면 안 됩니다.

---

## try4..

그래서 찾던 중 njs모듈 설치를 하면 된다고 해서 적용해봤습니다.

[Redirect to the encode url present in query parameter in NGINX](https://unix.stackexchange.com/a/628042)

1. 모듈 설치

```jsx
[root@victory-dev-106 conf.d]# sudo yum install nginx-module-njs
```

2. nginx.conf에 load_module 추가

```jsx
load_module modules/ngx_http_js_module.so;
```

3. http.js 생성

```jsx
function decoded_url(r) {
    return decodeURIComponent(r.args.url);
}

export default {decoded_url};
```

4. js로 파싱한 결과 set하는걸 http {} 블록 하위에 선언

```jsx
http {
    
    js_import http.js;
    js_set $decoded_url http.decoded_url;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen       8051;
        server_name  localhost;
        charset utf-8;
        rewrite_log on;

        location = / {
            if ($decoded_url ~ http[s]?:\/?\/?([^\/]+)(.*)$) {
                set $aaa $1;
                set $bbb $2;

                rewrite . $bbb break;
                proxy_pass http://$aaa;
            }
                proxy_redirect off;
                error_log /var/log/nginx/dn-m.img.error.log debug;
        }
		}
}
```

### 오프라인으로 njs 설치하고자 하면, nginx설치부터 njs포함시켜서 수동설치

[Download and install](https://nginx.org/en/docs/njs/install.html)

[https://jcartw.medium.com/how-to-install-nginx-from-source-with-njs-javascript-module-443e8e3a0cf2](https://jcartw.medium.com/how-to-install-nginx-from-source-with-njs-javascript-module-443e8e3a0cf2)

—>  대신 add-module이 configure된 것이라 nginx.conf에 load_module안해도 됨

### try4-problem

njs를 설치하면 인코딩된 url을 디코드시켜주기 때문에 문제가 해결됩니다.

하지만 운영중인 사이트에 모듈을 설치하기엔 부담이 있었습니다.

---

## try5..(solved)

좀더 검색해보니 아래 블로그를 발견!!

[https://lng1982.tistory.com/341](https://lng1982.tistory.com/341)

—> path나 body에 있는 string은 무조건 인코딩된다고 합니다.

그래서 저는 url을 header에 넣는 것으로 꼼수를 부려봤습니다.

feign으로 nginx호출할 때, url을 XAttachUrl 헤더에 셋팅하고 호출

[feign client class]

```java
@FeignClient(
    contextId = "client.ThirdPartyGatewayFileClient",
    name = "${crema.thirdparty.thirdparty-gateway.name}",
    configuration = {MultipartSupportConfig.class},
    primary = false
)
public interface ThirdPartyGatewayFileClient {
    @GetMapping(value = "attach/download")
    ResponseEntity<byte[]> download(@RequestHeader("XAttachUrl") String url);
}

```

- nginx쪽으로 [http://172.16.120.206:8051/attach/download](http://172.16.120.206:8051/attach/download)를 호출하면서 헤더에 url을 실어보냅니다.

[nginx.conf]

```bash
location ~* /attach/download {

    resolver 8.8.8.8;
   if ($http_XAttachUrl ~ http[s]?:\/?\/?([^\/]+)(.*)$) {
        set $part1 $1;
        set $part2 $2;
        rewrite . $part2 break;
        proxy_pass http://$part1;
    }

        proxy_redirect off;
        error_log /var/log/nginx/dn-m.img.error.log debug;
}
```

- $part1은 도메인
- $part2는 도메인 이후의 path, 파라미터 등의 나머지