---
layout: post
title: Could not locate PropertySource and the fail fast property is set, failing
date: 2022-04-20 22:02:00
last_modified_at : 2022-04-20 22:02:00
parent: Errors
has_children: false
nav_exclude: true
tags: [yml, spring cloud]
---

### error

```
java.lang.IllegalStateException: Could not locate PropertySource and the fail fast property is set, failing
        at org.springframework.cloud.config.client.ConfigServicePropertySourceLocator.locate(ConfigServicePropertySourceLocator.java:155) ~[spring-cloud-config-client-2.2.8.RELEASE.jar!/:2.2.8.RELEASE]
        at org.springframework.cloud.bootstrap.config.PropertySourceLocator.locateCollection(PropertySourceLocator.java:52) ~[spring-cloud-context-2.2.9.RELEASE.jar!/:2.2.9.RELEASE]
        at org.springframework.cloud.config.client.ConfigServicePropertySourceLocator.locateCollection(ConfigServicePropertySourceLocator.java:170) ~[spring-cloud-config-client-2.2.8.RELEASE.jar!/:2.2.8.RELEASE]
        at org.springframework.cloud.config.client.ConfigServicePropertySourceLocator$$FastClassBySpringCGLIB$$fa44b2a.invoke(<generated>) ~[spring-cloud-config-client-2.2.8.RELEASE.jar!/:2.2.8.RELEASE]
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218) ~[spring-core-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
Caused by: org.springframework.web.client.HttpServerErrorException$InternalServerError: 500 : [{"timestamp":"2022-04-19T23:09:15.574+00:00","status":500,"error":"Internal Server Error","message":"","path":"/config/crema/prod"}]
        at org.springframework.web.client.HttpServerErrorException.create(HttpServerErrorException.java:100) ~[spring-web-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.web.client.DefaultResponseErrorHandler.handleError(DefaultResponseErrorHandler.java:186) ~[spring-web-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.web.client.DefaultResponseErrorHandler.handleError(DefaultResponseErrorHandler.java:125) ~[spring-web-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.web.client.ResponseErrorHandler.handleError(ResponseErrorHandler.java:63) ~[spring-web-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.web.client.RestTemplate.handleResponse(RestTemplate.java:780) ~[spring-web-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:738) ~[spring-web-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:672) ~[spring-web-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.web.client.RestTemplate.exchange(RestTemplate.java:581) ~[spring-web-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.cloud.config.client.ConfigServicePropertySourceLocator.getRemoteEnvironment(ConfigServicePropertySourceLocator.java:271) ~[spring-cloud-config-client-2.2.8.RELEASE.jar!/:2.2.8.RELEASE]
        at org.springframework.cloud.config.client.ConfigServicePropertySourceLocator.locate(ConfigServicePropertySourceLocator.java:114) ~[spring-cloud-config-client-2.2.8.RELEASE.jar!/:2.2.8.RELEASE]
        ... 31 common frames omitted
```

### cause

yml파일에 오타가 있어서 발생했다.

암튼 yml파일을 읽어올수 없을 때 발생하는 것으로 location이 올바른지 profile에 맞는 yml인지 체크필요