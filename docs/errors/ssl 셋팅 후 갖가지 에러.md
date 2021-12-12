---
layout: default
title: ssl 셋팅 후 갖가지 에러
parent: Errors
has_children: false
nav_exclude: true
---

# ssl 셋팅 후 갖가지 에러

1. XMLHttpRequest cannot load https://aaa.ma.com:7443/. Response

to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin'

header is present on the requested resource. Origin

'http://127.0.0.1:8080' is therefore not allowed access.

- - 접근한 aaa.ma.com 주소는 헤더가 다른 http://127.0.0.1:8080을 ajax로 호출할 수 없다. cors 때문에
- - 공인인증서라면 인증서 설치에 문제겠지만 사설인증서인라면 인터넷브라우저의 인증서관리 화면에서 신뢰할수 잇는 루트기관으로 등록해준다
- - 사설 인증서 브라우저에서 저장하려면

(1) 인터넷옵션 > 내용 > 인증서 선택

(2) 신뢰할 수 있는 루트인증기관 탭 선택 > 가져오기

(3) 루트CA의 crt파일을 선택한다.

(4) 브라우저 재시작

2. 제우스에 ssl셋팅하고 재시작했는데..아래처럼 truststore가 없다고 나오면..키파일 위치를 잘못잡았거나, 키스토어가 이상하거나 ㅠㅜ

jeus.servlet.deployment.StartingException: File Not Found: D:\TmaxSoft\jeus6\config\testdomain\truststore

3. ERR_SSL_WEAK_SERVER_EPHEMERAL_DH_KEY

- 크롬 버전 : 45.0.2454.99에서 나오는 에러로 ssl셋팅했을 때 보안설정이 낮아서 그런것으로..
- 크롬으로 url쳐서 개발자모드 보면 콘솔에 아래처럼 찍히는데.. 먼가 이제까지 브라우저에서 커버된게 deprecated된건가.. 모르겠지만

**/deep/ combinator is deprecated. See https://www.chromestatus.com/features/6750456638341120 for more details.**

- 해결은 ssl인증서거는 설정에 ciper설정을 추가하면 된다.
- 만약 톰캣이라면 아래처럼 connector설정

<Connector ciphers="TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_DHE_RSA_WITH_AES_128_GCM_SHA256,TLS_DHE_DSS_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_SHA256,TLS_ECDHE_RSA_WITH_AES_128_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_SHA,TLS_ECDHE_RSA_WITH_AES_256_SHA384,TLS_ECDHE_ECDSA_WITH_AES_256_SHA384,TLS_ECDHE_RSA_WITH_AES_256_SHA,TLS_ECDHE_ECDSA_WITH_AES_256_SHA,TLS_DHE_RSA_WITH_AES_128_SHA256,TLS_DHE_RSA_WITH_AES_128_SHA,TLS_DHE_DSS_WITH_AES_128_SHA256,TLS_DHE_RSA_WITH_AES_256_SHA256,TLS_DHE_DSS_WITH_AES_256_SHA,TLS_DHE_RSA_WITH_AES_256_SHA" />

4. err_ssl_fallback_beyond_minimum_version

- 이것도 크롬에서만 나타난다
- 블로그) [http://ondemand.tistory.com/214](https://www.blogger.com/blog/post/edit/2689228726924373128/2895146181317602651#)

1) 크롬은 여전히 v45 에서 TLS v1.0 을 지원하고 있으니 TLS 버전 이슈는 아님, 2) 다만 크롬이 접속하려는 서버가 TLS Handshake 하는 과정에 이슈가 있을 경우, 기존에는 크롬이 Workaround 해주었지만, 3) v45 부터는 더이상 해당 Workaround 를 지원하지 않아서 발생하는 문제다 정도입니다. 구체적으로 어떤 웹 서버의 특정 버전이 이슈가 있는 것인지는 명확하게 정리된 내용을 찾지 못했습니다만 조치를 위해서는 서버측의 TLS Handshake 에 대한 보완이 필요한 것으로 추정