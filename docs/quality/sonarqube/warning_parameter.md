---
layout: post
title: Warning Parameter 'xxx' is never used
date: 2023-05-15 13:26:00
last_modified_at : 2023-05-15 13:26:00
parent: SonarQube
grand_parent: Quality Practice
nav_exclude: true
---

# Problem

controller에서 request parameter를 담아 service단을 호출하면서

annotation을 통해 사전 validation처리를 할 수 있는데요.

이 과정에서 로직에서는 사용하지 않는 argument지만, 

validation을 위해 annotation 클래스에 전달해야할 필요가 있을 수 있습니다.

이 때 unused warning이 발생하고, sonarqube에도 카운트되는 현상이 발생합니다.

# Code

## controller

```java
public class AController {
    private final AService aService;

    @PostMapping("/bot/{*aId}")
    public Mono<String> findAll(
            HttpServletRequest request,
            @PathVariable String aId,
            @Valid @RequestBody Payload payload) {
        return Mono.fromSupplier(() -> aService.search(request, aId, payload)).doOnNext(log::debug).log();
    }
}
```

- HttpServletRequest를 통해 얻어올 수 있지만, path variable을 통해 aId를 받아서 전달해봅니다.

## service

```java
public class AService {

    @AccessValidate
    public String search(HttpServletRequest request, String aId, Payload payload) {
		}
}
```

- AccessValidate custom annotation에서 aId값으로 유효한 요청인지 검사하는 로직입니다.

# Try1. deal with logger

- 로깅에 추가하면, unused는 피할 수 있습니다.

# Try2. add variable name prefix - `ignored~`

- argument 변수명에 ignored*로 시작하게 변경하는 방법이 있습니다.
- 대신 annotation 클래스에도 동일하게 ignored로 변경해야합니다.

```java
public class AService {

    @AccessValidate
    public String search(HttpServletRequest request, String ignoredAId, Payload payload) {
		}
}
```