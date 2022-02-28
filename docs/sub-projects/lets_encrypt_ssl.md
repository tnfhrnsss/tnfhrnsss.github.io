---
layout: post
title: let's encrypt ssl
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
# parent: Sub Projects
parent: Home
has_children: false
nav_order: 3
nav_exclude: true
---

# let's encrypt with (window, springboot)

로컬(윈도우)에서 springboot 어플리케이션에 ssl을 추가해서 테스트해볼 일이 생겼다.

무료 ssl로 let's encrypt를 활용하기로 하고 설치 및 적용해봄

1. let's encrypt는 certbot을 이용해서 설치하면 된다. 
2. 아래 페이지로 이동해서 certbot install파일을 내려받는다. ([다운로드](https://dl.eff.org/certbot-beta-installer-win32.exe))

[Certbot Instructions](https://certbot.eff.org/instructions?ws=other&os=windows)

1. 관리자 권한으로 cmd창을 열고 아래 명령어를 쳐서 잘 설치된건지 확인한다.

```bash
C:\WINDOWS\system32> certbot --help
```

1. dns 등록

freenom 이용함 (무료 dns)

1. dns까지 준비되었다면, ssl을 생성한다. (dns가 없다면, 중간에 실패난다.)

```bash
C:\WINDOWS\system32>certbot certonly --standalone
Saving debug log to C:\Certbot\log\letsencrypt.log
Please enter the domain name(s) you would like on your certificate (comma and/or
space separated) (Enter 'c' to cancel): ~~~~~.~~~~ (dns주소 입력)
Requesting a certificate for ~~~~~.~~~~

Successfully received certificate.
Certificate is saved at: C:\Certbot\live\~~~~~.~~~~fullchain.pem
Key is saved at:         C:\Certbot\live\~~~~~.~~~~\privkey.pem
This certificate expires on 2022-02-17.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.
```

참조) 자세한 내용은 아래 블로그 참고

[SpringBoot - Let's Encrypt로 무료 SSL인증서를 발급받아 SpringBoot에 적용하기(Https)](https://galid1.tistory.com/612)

6. 생성한 ssl을 윈도우 폴더에 위치시키고 yml파일을 수정한다.

```yaml
server:
  port: 19010
  ssl:
        enabled: true
        key-store: ~~~.jks
        key-store-type: "JKS" 또는 "PKCS12" 지정 
        key-store-password: ~~~~~~~~ 인증서 암호
        key-alias: 선택항목으로 alias또는 CN명
        trust-store: 선택항목
        trust-store-password: 선택항목
```

7. 그리고 @SpringBootApplication파일을 run시킨다.

8. 내부망에 있다보니, lte로 테스트했을때, 유효한 인증서로 보였는데, line developers의 webhook url로 등록하고 verify하니까 아래 에러가 발생했다.

```
An SSL connection error occurred. 
Confirm that your webhook URL uses HTTPS and has an SSL/TLS certificate issued by 
a root certification authority that's widely trusted by most browsers
```

뭔가 유효하지 않은 느낌이다.

9. 인증서 유효성 체크사이트([https://www.digicert.com/help/](https://www.digicert.com/help/)) 에서 체크해본다. —> 유효하지 않다고 나옴.

1. 브라우저에서 ip로 호출해보니 "ERR_CERT_COMMON_NAME_INVALID"라고 나온다. —> 발급대상과 일치하지 않다는 에러라고 한다.
2. 포트를 19010에서 443으로 변경하고 다시 digicert에서 검사돌려보니 "TLS Certificate is not trusted" 라고 나온다. —> 인증서 체인을 잘못 건것으로 keytool -list -v -keystore keystore.jks실행시켜서 인증서 개수가 3개 인지 확인한다.