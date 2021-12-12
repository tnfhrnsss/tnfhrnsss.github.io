---
layout: default
title: Please log in (using, e.g., "su") as the (unprivileged) user that will
parent: Errors
has_children: false
nav_exclude: true
---

# Please log in (using, e.g., "su") as the (unprivileged) user that will

[현상]

Please log in (using, e.g., "su") as the (unprivileged) user that will

[원인]

유닉스에 postgresql을 설치하고 나서 su로 로긴해서 ./pg_ctl로 서비스를 시작하려하는데 저런 에러가...

일본 구글에서 해답을 찾았다.

>> 블로그 [http://nobuneko.com/blog/archives/2011/05/postgresqlinitdb_cannot_be_run.html](https://www.blogger.com/blog/post/edit/2689228726924373128/8436481802871656554#)

>> 해석

솔라리스 등에서 initdb를 실행하려고 하면,  initdb: cannot be run as root라고 하는 에러메세지가 표시되고 initdb를 실행시킬 수 없게 된다

에러메세지를 읽으면, initdb는 root유저로 실행시킬 수 없다는 것을 알게 된다.

이 에러를 해결하려면 root 유저 외의 사용자로 initdb를 실행시키면 된다.

예를 들면, su - postgres에서 postgres유저를 만드니까 initdb를 postgres로 실행시키면된다.

같은 케이스는 아니지만 나도 su - postgres로 올리니까 해결됨.