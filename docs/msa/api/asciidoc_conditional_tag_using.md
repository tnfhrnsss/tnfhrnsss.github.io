---
layout: post
title: asciidoc conditional tag using
date: 2022-07-04 21:37:00
last_modified_at : 2022-07-04 21:37:00
parent: Api
grand_parent: Msa
nav_exclude: true
tags: [asciidoc]
---

## problem

사내에서 api doc으로 asciidoc을 쓰고 있는데요.

일부 api들은 파라미터 정보나 보안요건 등을 기술하고 있고, 일부 api들은 헤더정보등이 표시되어야 하는 등 

include되는 adoc파일의 그룹핑? 이 필요했습니다.

## try

검색해보니 별도로 함수를 만들지 않는 한, 불가능해 보였는데요

tag기능을 쓰면 될 듯해서 적용해봤습니다.

[api.adoc]

```java
=== findAll

Center 목록을 가져오는 API 입니다.

:class-path: {snippets}/CenterResourceTest
:api-name: findAll
:tag: permissions
include::{include-template-dir}/include-template.adoc[]
```

[include-template.adoc]

```java
ifeval::[{tags} == "permissions"]
include::{class-path}/{api-name}/required-permissions.adoc[leveloffset=+1,opts=optional]
endif::[]

include::{class-path}/{api-name}/curl-request.adoc[leveloffset=+1,opts=optional]
include::{class-path}/{api-name}/http-request.adoc[leveloffset=+1,opts=optional]

ifeval::[{tag} == "headers"]
include::{class-path}/{api-name}/request-headers.adoc[leveloffset=+1,opts=optional]
endif::[]
```

## reference

[AsciiDoc - Conditionals](https://docs.asciidoctor.org/asciidoc/latest/directives/conditionals/)

[AsciiDoc - Include Content by Tagged Regions](https://docs.asciidoctor.org/asciidoc/latest/directives/include-tagged-regions/#tagging-regions)