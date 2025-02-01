---
layout: post
title: mac에서 putty로 window 터널링
date: 2025-02-02 00:03:00
last_modified_at : 2025-02-02 00:03:00
parent: Etc
has_children: false
nav_exclude: true
tags: [mac, putty]
---


- 맥은 putty가 필요없다. 터미널을 사용한다.
- 윈도우에서 사용한 ppk파일은 사용할 수 없다. 
- 윈도우에서 puttygen으로 해당 ppk를 불러와서 메뉴에 보면 Conversions>Export OpenSSH key가 있다. 이걸로 변환하면 맥에서 사용가능
- 맥에 해당 키를 두고, 키 파일에 권한을 줘야 한다 chmod 400 key이름
- 터미널을 열어서 명령어로 포트를 연다.
```
marsais$ ssh -i mackey [터널링할 계정id@서버](mailto:로그인도메인 이메일주소) -p 22 -N -L 3000:목적지 로컬 공인IP주소:3389
```
- mackey는 export한 개인키이다
- p는 7에서 적은 서버에 접속할 포트를 적는다. 위에 22포트로 접속하고자 22로 넣었다.
- N은 원격 쉘을 실행시키지 않고 접속만 하려는 옵션
- L은 터널링으로 붙을 pc정보를 적을 옵션으로 뒤에
- 터널링으로 열 포트:접속할ip:접속할pc의 포트 --> 난 3389포트로 내 pc에 3000포트로 열어둘것이다
- 그래서 맥 앱스토어에 가면 microsoft remote desktop앱 설치
- microsoft remote desktop > pc name에 127.0.0.1:3000으로 저장하고 아이디와 비번에 내 pc정보 넣는다
- 접속한다.
- 만약, 윈도우용 ppk로 붙을라고 하면, Bad passphrase, try again for 에러가 난다.