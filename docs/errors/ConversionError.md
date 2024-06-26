---
layout: post
title: ConversionNotSupportedException
description: ConversionNotSupportedException
date: 2022-03-27 12:58:45
last_modified_at : 2022-03-27 12:58:45
parent: Errors
has_children: false
nav_exclude: true
tags: [java8, annotation]
keywords: java8
---

#### error log
expressed through field 'expirations'; nested exception is org.springframework.beans.ConversionNotSupportedException: Failed to convert value of type 'java.lang.String' to required type 'java.util.List'; nested exception is java.lang.IllegalStateException:

Cannot convert value of type 'java.lang.String' to required type 'DeviceExpiration':

no matching editors or conversion strategy found

설정yml

```
login:
  expirations:
    - device: pc
      limit: 5
    - device: mobile
      limit: 1440
    - device: tablet
      limit: 1440
```

#### cause

```
@Value("${login.expirations:}")
private List<DeviceExpiration> expirations;
```

#### solution
To mapping Object, using `@ConfigurationProperties`로 선언해야한다.

```
@Getter
@Component
@ConfigurationProperties("login")
public class ExpirationConfig {
    private List<DeviceExpiration> expirations;

    public void setExpirations(List<DeviceExpiration> expirations) {
        this.expirations = expirations;
    }
}
```

이렇게 하니 된다