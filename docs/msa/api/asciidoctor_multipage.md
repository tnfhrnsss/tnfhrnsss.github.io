---
layout: post
title: asciidoctor multiple pages from a single page
date: 2022-04-02 16:06:00
last_modified_at : 2022-04-02 16:06:00
parent: Api
grand_parent: Msa
nav_exclude: true
tags: [asciidoc, asciidoctor]
---

### check asciidoctor extension

먼저 extension이 있는지 체크했다. 

[https://asciidoctor.org/docs/extensions/](https://asciidoctor.org/docs/extensions/)

### exist asciidoctor-multipage plugin

**[asciidoctor-multipage](https://github.com/owenh000/asciidoctor-multipage)**

exampe page: [https://owenh.net/static/portfolio/nxlog/guide/nxlog-user-guide.html](https://owenh.net/static/portfolio/nxlog/guide/nxlog-user-guide.html)

but ruby base.

### asciidoctor multipage maver example

So, I made a maven version project.

[https://github.com/tnfhrnsss/asciidoctor-multipage-maven-example](https://github.com/tnfhrnsss/asciidoctor-multipage-maven-example)

![asciidoctor1](../img/asciidoctor1.png)

```
maven clean install
```

하면, 

기존에는 index.html 페이지 한개로 렌더링되던 부분이

각각의 페이지로 구성

### reference

[https://discuss.asciidoctor.org/Using-the-multi-page-converter-from-maven-td8549.html](https://discuss.asciidoctor.org/Using-the-multi-page-converter-from-maven-td8549.html)

[https://rubygems.org/gems/asciidoctor-multipage](https://rubygems.org/gems/asciidoctor-multipage)

[https://github.com/asciidoctor/asciidoctor-maven-examples](https://github.com/asciidoctor/asciidoctor-maven-examples)에서 revealjs가 maven-gem구성