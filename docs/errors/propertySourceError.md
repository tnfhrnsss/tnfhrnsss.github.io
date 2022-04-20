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
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:779) ~[spring-aop-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163) ~[spring-aop-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:750) ~[spring-aop-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.retry.interceptor.RetryOperationsInterceptor$1.doWithRetry(RetryOperationsInterceptor.java:91) ~[spring-retry-1.2.5.RELEASE.jar!/:na]
        at org.springframework.retry.support.RetryTemplate.doExecute(RetryTemplate.java:287) ~[spring-retry-1.2.5.RELEASE.jar!/:na]
        at org.springframework.retry.support.RetryTemplate.execute(RetryTemplate.java:164) ~[spring-retry-1.2.5.RELEASE.jar!/:na]
        at org.springframework.retry.interceptor.RetryOperationsInterceptor.invoke(RetryOperationsInterceptor.java:118) ~[spring-retry-1.2.5.RELEASE.jar!/:na]
        at org.springframework.retry.annotation.AnnotationAwareRetryOperationsInterceptor.invoke(AnnotationAwareRetryOperationsInterceptor.java:153) ~[spring-retry-1.2.5.RELEASE.jar!/:na]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:750) ~[spring-aop-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:692) ~[spring-aop-5.2.15.RELEASE.jar!/:5.2.15.RELEASE]
        at org.springframework.cloud.config.client.ConfigServicePropertySourceLocator$$EnhancerBySpringCGLIB$$537fc30a.locateCollection(<generated>) ~[spring-cloud-config-client-2.2.8.RELEASE.jar!/:2.2.8.RELEASE]
        at org.springframework.cloud.bootstrap.config.PropertySourceBootstrapConfiguration.initialize(PropertySourceBootstrapConfiguration.java:98) ~[spring-cloud-context-2.2.9.RELEASE.jar!/:2.2.9.RELEASE]
        at org.springframework.boot.SpringApplication.applyInitializers(SpringApplication.java:623) [spring-boot-2.3.12.RELEASE.jar!/:2.3.12.RELEASE]
        at org.springframework.boot.SpringApplication.prepareContext(SpringApplication.java:367) [spring-boot-2.3.12.RELEASE.jar!/:2.3.12.RELEASE]
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:311) [spring-boot-2.3.12.RELEASE.jar!/:2.3.12.RELEASE]
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1247) [spring-boot-2.3.12.RELEASE.jar!/:2.3.12.RELEASE]
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1236) [spring-boot-2.3.12.RELEASE.jar!/:2.3.12.RELEASE]
        at spectra.attic.talk.crema.CremaApplication.main(CremaApplication.java:22) [classes!/:na]
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_292]
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_292]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_292]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_292]
        at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:49) [crema-boot-3.1.0-exec.jar:na]
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:108) [crema-boot-3.1.0-exec.jar:na]
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:58) [crema-boot-3.1.0-exec.jar:na]
        at org.springframework.boot.loader.PropertiesLauncher.main(PropertiesLauncher.java:467) [crema-boot-3.1.0-exec.jar:na]
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