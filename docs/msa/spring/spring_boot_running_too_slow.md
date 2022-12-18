---
layout: post
title: spring boot application running too slow
date: 2022-12-18 15:12:00
last_modified_at : 2022-12-18 15:12:00
parent: Spring
grand_parent: Msa
nav_exclude: true
---

# problem

- build, install, load도 느리지만,
- spring boot start가 제일 느리다.
    - 최고 800초를 넘는다.
    - 평균 500~600초

# Cause

## Try1.

1. host에 127을 추가하라는 글 (x)→ 이미 있음
2. plugin이 문제일 수 있다고 해서 안쓰는 플러그인 disable처리(x)
3. hibernate validator를 사용안함으로 설정하라는 글 (x)
    1. application.yml에 아래 설정 추가
    
    ```yaml
    spring:
      jpa:
        properties:
          javax:
            persistence:
              validation:
                mode: none
    ```
    
4. start할때 before build옵션 제거
5. run java를 java8에서 java17로 변경 - 아주 약간 빨라짐

## Try2. add java option

- disable launch optimization 추가
- disable jmx agent 옵션 추가

![spring_boot_1](../img/spring_boot_1.png)

- result
    - as-is : 평균 550초
    - to-be: 평균 400초

## Try3. change server (tomcat → undertow)

- pom.xml 변경
    
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-undertow</artifactId>
    </dependency>
    ```
    
- application.yml 변경
    
    ```yaml
    server:
      undertow:
        threads:
          io: 8
          worker: 200
    ```
    
- result
    - 454 ~ 471초
    - 경량이라 기대했는데, 빨라지지 않음

## Try4.

## Reference

[https://stackoverflow.com/a/59654120/14257397](https://stackoverflow.com/a/59654120/14257397)