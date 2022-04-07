---
layout: post
title: UnixFileSystem.createFileExclusively
date: 2022-04-07 21:36:00
last_modified_at : 2022-04-07 21:36:00
parent: Errors
has_children: false
nav_exclude: true
tags: [unix, file]
---

### problem

- apache poi로 엑셀 파일을 생성해서 unix시스템에 파일쓰기할 때 Permission denied 발생

### error

![unixfile1](../img/unixfile1.png)

```
java.io.IOException: Permission denied
at java.io.UnixFileSystem.createFileExclusively(Native Method)
at java.io.File.createTempFile(File.java:2063)
at org.apache.poi.util.DefaultTempFileCreationsStrategy.createTempFile(DefaultTempFileCreationsStrategy.java:110)
```

### solved

- 만약에 톰캣 기본 폴더를 지정하지 않았다면, /tmp 경로에 가면 아래 폴더들이 존재한다.

![unixfile2](../img/unixfile2.png)

- hsperfdata_로 시작하는 폴더와 poifiles폴더의 권한이 맞게 되어있는지 체크
- 또는 기본 폴더도 변경하자 (centos에서는 주기적으로 /tmp 하위 삭제함)

```
server:
  tomcat:
    basedir: /tmp 외의 폴더로 변경
```

### reference

[java.io.IOException: Permission denied](https://us-misonam.tistory.com/53)