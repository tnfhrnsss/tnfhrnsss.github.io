---
layout: post
title: java.lang.IllegalStateException...
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
---

# java.lang.IllegalStateException: RequestParam.value() was empty on parameter 0

원인 : FeignClient로 선언한 adapter에 @RequestParam으로 값을 받아오는 부분에 맵핑을 잘못함.

기존>

@RequestParam(defaultValue ="0", required = false) int offset,

@RequestParam(defaultValue = "10", required = false) int limit

변경>

@RequestParam("offset") int offset,

@RequestParam("limit") int limit