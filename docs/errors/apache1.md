---
layout: post
title: BrowserMatch
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
---

# BrowserMatch

기본 아파치를 윈도우에 설치하고 나니 httpd-ssl.conf에

아래 설정이 있었다.

- --------------------

BrowserMatch ".*MSIE.*" \nokeepalive ssl-unclean-shutdown \downgrade-1.0 force-response-1.0

- --------------------

콘솔에 로그인해서 상담하기까지 활성화하는데 1분이나 걸리게 되어서

주석을 하고 재시작하니, 엄청 빨라짐.

파악해본결과 이 블로그에서 (http://blogs.msdn.com/b/ieinternals/archive/2011/03/26/https-and-connection-close-is-your-apache-modssl-server-configuration-set-to-slow.aspx)

Four years ago, there was a[public call](https://www.blogger.com/blog/post/edit/2689228726924373128/658335713030481387#)to update the guidance to reflect the fact that users of more modern browsers were paying an unneeded performance penalty. Finally, in June 2010, the default guidance was changed in recognition of the fact that the problem never affected IE6 and later:

> BrowserMatch ".*MSIE [1-5].*" \nokeepalive ssl-unclean-shutdown \downgrade-1.0 force-response-1.0
> 

Unfortunately, many major Apache installations still haven’t been updated with even this guidance. Also, alert readers will spot a very obvious problem with the “new” regular expression.

In the expression above, any IE version that *starts with*“1” will be treated as outdated and served connection slowly without Keep-Alive. Internet Explorer 1.0 didn’t even support SSL at all ([SSL was added in 2.0](https://www.blogger.com/blog/post/edit/2689228726924373128/658335713030481387#)), but worse, this loosely-written regular expression will also match*future***MSIE 10.0,MSIE 11.0,MSIE 12.0**(etc)****user-agent strings. Hence, Apache hosts will one day find that the*newest*browsers are forced into the “slow” lane!

At the very least, Apache hosts should update their regular expression to this:

> BrowserMatch ".*MSIE [2-5]\..*" \nokeepalive ssl-unclean-shutdown \downgrade-1.0 force-response-1.0
> 

…but ultimately, they should probably remove this hack altogether. The ancient Internet Explorer 6’s marketshare is in decline, and there’s almost never any business reason to try to accommodate even older browsers.

요약하면 삭제해야할 설정인데 아파치에서 아직도 껴서 배포하고 있다는 것..

- --------------

원래 BrowserMatch로 사용할 수 있는 유요한 설정은 특정 버전의 브라우저에 문제가 있어서 아래처럼 분기하고자 할때 쓴다.

(http://webdir.tistory.com/178)

BrowserMatch "Mozilla/2" nokeepalive

### BrowserMatch "MSIE 4\.0b2;" nokeepalive downgrade-1.0 force-response-1.0

### BrowserMatch "RealPlayer 4\.0" force-response-1.0

### BrowserMatch "Java/1\.0" force-response-1.0

### BrowserMatch "JDK/1\.0" force-response-1.0

BrowserMatch 지시자는 특정 브라우저들에 대한 특정 수행을 지시하기 위한 설정들이다.

- 첫번째 것은 네스케이프 2.x 또는 그를 흉내내는 브라우저에 대하여 KeepAlive 기능을 쓰지 않도록 한다. 이 브라우저들은 KeepAlive 구현에 문제점을 갖고 있기 때문이다.
- 두번째 것은 HTTP/1.1을 잘못 구현하였고 301 또는 302 (redirect)반응에 대하여 KeepAlive를 제대로 지원하지 못하는 마이크로소프트 인터넷 익스플로러 4.0b2를 위한 것이다.
- 세번째 것은 네번째 다섯번째 것들은 기본적인 1.1 반응도 제대로 처리하지 못함으로써 HTTP/1.1 스펙을 위반하고 있는 브라우저에 대하여 HTTP/1.1 반응을 하여 하지 않도록 한다.

### BrowserMatch "Microsoft Data Access Internet Publishing Provider" redirect-carefully

### BrowserMatch "MS FrontPage" redirect-carefully

### BrowserMatch "^WebDrive" redirect-carefully

### BrowserMatch "^WebDAVFS/1.[0123]" redirect-carefully

### BrowserMatch "^gnome-vfs/1.0" redirect-carefully

### BrowserMatch "^XML Spy" redirect-carefully

### BrowserMatch "^Dreamweaver-WebDAV-SCM1" redirect-carefully

위 지시자들도 구현에 문제가 있는 것들에 대한 처리방식을 지정한 것이다.