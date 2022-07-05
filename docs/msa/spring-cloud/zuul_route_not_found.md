---
layout: post
title: zuul routes 404 not found error
date: 2022-07-05 21:20:00
last_modified_at : 2022-07-05 21:20:00
parent: SpringCloud
grand_parent: Msa
nav_exclude: true
tags: [spring cloud, zuul]
---

## 요구사항

웹/모바일 프론트에서 호출하는 백엔드API는 test서비스의 rest로 시작하는 API로만 접근하도록 한다.

[https://stackoverflow.com/questions/12569308/spring-difference-of-and-with-regards-to-paths](https://stackoverflow.com/questions/12569308/spring-difference-of-and-with-regards-to-paths)

eureka에 등록된 serviceId를 test라고 했을 때

## AS-IS

기존에는 아래 설정으로 프론트엔드에서 [http://localhost:8080/test/users](http://localhost:8080/test/users)라는 API를  호출했다면

```yaml
zuul:
  routes:
    test:
      path: /test/**
```

## TO-BE

[http://localhost:8080/test/](http://localhost:8080/test/users)rest/users 처럼 특정 프론트에서 호출하는 API는 /rest하위의 리소스로만 접근하도록 해야한다.

### try..

```yaml
zuul:
  routes:
    test:
      path: /test/rest/**
```

- 이렇게 변경하면, 일반 requestmapping의 경우는 동작하는데, pathVariable로 path에 변수가 들어가게 되면 **404 not found** 에러가 난다.

```java
@RequestMapping("rest/centers/{centerId}/receptions")
```

## solution

```yaml
zuul:
  routes:
    test:
      path: /rest/**
```

이유는 알수 없지만, 뎁스로 체크가 안되는 것 같다. 관련 reference도 못찾았다.

## addition

사실 원하는 것은 restapi는 /test/rest/하위의 API로만 접근하도록 하고 싶었다.

zuul에 routes를 추가하고 ignorePattern을 설정하는 것으론 불가능했다. (/test하위로 접근하면 안되고 /test/rest하위로만 접근되도록 하는 것)

그래서 서비스에서 보안요건으로 authority를 체크하는 기능이 있었는데, 거기에 허용 path로 설정하는 방식으로 작업을 했다.

그래서 접근불가한 path는 403 에러로 뱉게 했다. 

(zuul의 ignorePattern이나 ignoreServices로 설정하면 404 에러로 되므로 403이 맞는 것 같다.)

## reference

[https://cloud.spring.io/spring-cloud-netflix/multi/multi__router_and_filter_zuul.html](https://cloud.spring.io/spring-cloud-netflix/multi/multi__router_and_filter_zuul.html)