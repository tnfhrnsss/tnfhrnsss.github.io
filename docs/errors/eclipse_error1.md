---
layout: post
title: Unknown Faceted Project Problem
date: 2015-07-08 22:51:00
last_modified_at : 2022-07-08 22:51:00
parent: Errors
has_children: false
nav_exclude: true
tags: [eclipse]
description: Unknown Faceted Project Problem
keywords: eclipse
---

## problem

이클립스에서 아래처럼 나오면

```prolog
Description     Resource     Path     Location     Type

Java compiler level does not match the version of the installed Java project facet.
webapps Unknown Faceted Project Problem (Java Version Mismatch)
```

## solution

## try1

properties > project facet > java버전을 낮추는데

## try2

dynamic web module버전이 3.0에서 2.5로 안내려간다면

.setting> > org.eclipse.wst.common.project.facet.core.xml

파일을 열어 jst.web을 2.5로 변경하거나 컴파일 버전을 바꾼다