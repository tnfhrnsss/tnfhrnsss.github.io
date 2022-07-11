---
layout: post
title: webpage 문자를 스크랩핑해서 갱신여부 체크하는 어플리케이션 개발
date: 2022-07-11 22:30:00
last_modified_at : 2022-07-11 22:30:00
parent: Etc
has_children: false
nav_exclude: true
tags: [scrapping, python]
---

## 요건

서드파티 api 페이지의 버전이 안올라가도 가끔 변경되고 있음. 이력을 알 수 없어서

스크랩핑해서 변경된 이력을 일단위로 체크하는 것 필요

## 기술검토

**Web Scraping vs Web Crawling**

[https://stackhoarder.com/2019/08/18/python부터-web-scraping-까지-최단-시간에-익혀보자/](https://stackhoarder.com/2019/08/18/python%EB%B6%80%ED%84%B0-web-scraping-%EA%B9%8C%EC%A7%80-%EC%B5%9C%EB%8B%A8-%EC%8B%9C%EA%B0%84%EC%97%90-%EC%9D%B5%ED%98%80%EB%B3%B4%EC%9E%90/)