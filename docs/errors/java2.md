---
layout: post
title: Unsatisfied dependency expressed through constructor parameter 8; nested exception...
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
---

# Unsatisfied dependency expressed through constructor parameter 8; nested exception...

오류 : Unsatisfied dependency expressed through constructor parameter 8; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'TestAdapter': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: Method find not annotated with HTTP method type (ex. GET, POST)

원인:TestAdapter에 부모가 선언하지 않은 이름의 메소드로 선언되어있어서 그러함. 즉 부모에 있는데TestAdapter에 구현체가 없어서 나오는 에러