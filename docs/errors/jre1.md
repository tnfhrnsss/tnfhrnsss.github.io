---
layout: post
title: 애플 Symantec Root 인증서 제거로 인한 인증서 갱신
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
---

# 애플 Symantec Root 인증서 제거로 인한 인증서 갱신

에러 : PKIX path building failed

원인

- 애플에서 symantec root인증서 제거해서, 인증불가 발생
- JRE 1.8.0_101 미만인 경우 발생

조치

- JRE 1.8.0_101 이상으로 버전업
- 인증서 추가

[Java - PKIX path building failed](https://blog.geunho.dev/posts/pkix-path-building-failed/)