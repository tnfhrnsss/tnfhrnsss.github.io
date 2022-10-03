---
layout: post
title: spring config additional-location not working
date: 2022-07-19 20:05:00
last_modified_at : 2022-07-19 20:05:00
parent: Spring
grand_parent: Msa
nav_exclude: true
tags: [spring cloud, config, yml]
---

## to do..

변경가능성이 있는 외부 접속정보는 git에 push하고 있지 않고 있어서

품질 서버에서 테스트할 때마다 접속정보를 yml에 설정해야 하는 불편함이 있었다.

접속 정보에 해당하는 property만 별도의 yml파일에 정의하고

application 띄울 때 덮어씌워지게 한다.

## problem

```bash
java -Xms1024m -Xmx1024m -Dspring.profiles.active=oracle -Dspring.config.additional-location=file:/module/crema.yml -jar crema-boot-4.0.0.DEV-SNAPSHOT-exec.jar
```

module폴더에 별도의 yml을 정의해서 띄웠는데, override되지 않았다..

안되는 원인을 한참을 찾았다..

spring버전부터 먼저 체크해서 doc을 봤어야했다….

## solution

### spring boot 2.3.x

```bash
spring.cloud.config.allowOverride: true
spring.cloud.config.overrideNone: true
```

결론은 spring cloud설정을 해줘야 한다.

해당 설정은 설정파일이 remote로 가져와야할 때 해주는 설정인데 상세 옵션은 아래와 같다.

yml도 naming이 중요하다고 느껴지는 부분이다. (설정 key명칭이 기능이랑 매치가 안되는 건 나만의 생각인지 모르겠다)

`spring.cloud.config.allowOverride`

- true이면 override (기본값)

`spring.cloud.config.overrideNone`

- true이면 모든 로컬 `PropertySource`가 Remote Properties로 덮어쓰게 됨 (기본값 false)

`spring.cloud.config.overrideSystemProperties`

- false이면 시스템 환경설정도 override된다 (기본값 true)

### spring boot 2.4.x

2.4버전 이후는 config 우선순위 설정이 변경되었다. option도 생기고 ..

## reference

[[Spring] Spring Cloud Config 설정 파일 적용](https://jjeong.tistory.com/1489)

[[SPRING BOOT] 외부 설정 및 우선순위 (Externalized Configuration and priorities)](https://velog.io/@ashappyasikonw/SPRING-BOOT-%EC%99%B8%EB%B6%80-%EC%84%A4%EC%A0%95-%EB%B0%8F-%EC%9A%B0%EC%84%A0%EC%88%9C%EC%9C%84-Externalized-Configuration-and-priorities)

[What is the loading precedence for properties from Spring Cloud Config?](https://stackoverflow.com/a/62838334)

[Spring Cloud](https://shortstories.gitbook.io/studybook/spring_ad00_b828_c815_b9ac/cloud#remote-properties)