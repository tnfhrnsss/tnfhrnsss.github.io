---
layout: post
title: spring boot admin metrics for cache
date: 2022-10-05 20:51:00
last_modified_at : 2022-10-05 20:51:00
parent: Spring
grand_parent: Msa
nav_exclude: true
tags: [spring, admin, metrics, cache]
---

## problem

- 코드를 보면, insights > Details 메뉴에 cache에 대한 지표가 표시되어야 하는데. 표시가 안되고 있어서 코드를 따라가다보니 cache 지표값을 actuator/metrics에서 뽑아오는 것인데
- 그래서 아래 cache.gets url로 호출해봤는데 404에러가 나고 있었습니다.
    
    ```yaml
    http://localhost:9010/actuator/metrics/cache.gets
    ```
    
- 찾아보니 마이크로미터에서 지원하는 지표에는 cache가 없었는데요.. 그래서 안나오는게 당연
    
    ```yaml
    AppOptics
    Atlas
    Datadog
    Dynatrace
    Elastic
    Ganglia
    Graphite
    Humio
    Influx
    JMX
    KairosDB
    New Relic
    Prometheus
    SignalFx
    Simple (in-memory)
    StatsD
    Wavefront
    ```
    

## cause

- 서비스 시작 시점에 사용할 수 있는 캐시가 있으면 메트릭에 등록되기 때문에, 그 시점 이후에 생성되는 캐시는 추가 코드를 통해서 등록해줘야한다고 합니다.

## solved

- CacheMetricsRegistrar을 상속받아서 구현하면 된다고 함! ([https://gunju-ko.github.io/spring/2018/12/19/SpringMircometer.html](https://gunju-ko.github.io/spring/2018/12/19/SpringMircometer.html))
- 해보진 않았지만, [https://medium.com/@iliamsharipov_56660/spring-boot-actuator-for-concurrentmapcache-2c7f0d290934](https://medium.com/@iliamsharipov_56660/spring-boot-actuator-for-concurrentmapcache-2c7f0d290934) 를 참고하면 적용가능해보입니다.

## reference

[Spring Boot Actuator - 마이크로미터 지원](https://gunju-ko.github.io/spring/2018/12/19/SpringMircometer.html)

[Spring Boot Actuator Web API Documentation](https://docs.spring.io/spring-boot/docs/current/actuator-api/htmlsingle/#metrics)

[https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)