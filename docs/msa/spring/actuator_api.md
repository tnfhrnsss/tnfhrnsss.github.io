---
layout: post
title: how to use actuator api 
date: 2023-05-28 12:37:00
last_modified_at : 2023-05-28 12:37:00
parent: Spring
grand_parent: Msa
nav_exclude: true
---

# actuator로 logger 변경

```java
curl -L 'http://localhost:7000/actuator/loggers/{package path}' \
-H 'Content-Type: application/json' \
-d '{"configuredLevel": null, "effectiveLevel": "DEBUG"}'
```