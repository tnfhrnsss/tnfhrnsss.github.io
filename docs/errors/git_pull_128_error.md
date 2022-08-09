---
layout: post
title: git pull > exit code 128 error
date: 2022-08-09 20:02:00
last_modified_at : 2022-08-09 20:02:00
parent: Errors
has_children: false
nav_exclude: true
tags: [git]
---

## error log

```java
git.exe clone --progress --branch develop -v "[https://--/qm/loadtest/tree/develop](https://--/qm/loadtest/tree/develop)" "D:\_GIT\attic-project\qm\develop"
Cloning into 'D:\_GIT\attic-project\qm\develop'...
fatal: unable to update url base from redirection:
asked for: [https://--/qm/loadtest/tree/develop/info/refs?service=git-upload-pack](https://--/qm/loadtest/tree/develop/info/refs?service=git-upload-pack)
redirect: [https://--/users/sign_in](https://gitlab.spectra.co.kr/users/sign_in)

git did not exit cleanly (exit code 128) (391 ms @ 2022-08-01 오전 8:41:19)
```

## solution

[Gitlabでpushしようとしたら、fatal: unable to update url base from redirection: と出てpushできない](https://teratail.com/questions/258703)

에 댓글을 보면 기존 브랜치를 삭제하고 새로 받으라 되어있는데

그걸로도 잘 안되어서

그냥 새로운 폴더에서 pull하니 잘되었다~