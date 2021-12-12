---
layout: default
title: Source files should not have any duplicated blocks
parent: SonarQube
grand_parent: Quality Practice
nav_exclude: true
---

# Source files should not have any duplicated blocks

## **1 duplicated blocks of code must be removed.**

**[원인]**

block자체가 중복되는 코드들이 존재할 때 발생

![img.png](../img/img.png)

의존성이 없는 컴포넌트였지만, 한개의 서비스안에 위치했기 때문에 발생

**[해결]**

중복되는 파일 중 한개를 열어서 왼쪽 회색 부분에 마우스 오버하면 duplicated된 block을 확인할 수 있다.

![img1.png](../img/img1.png)

해결 방법은 여러가지가 되겠지만, 중복적인 코드가 필요한 이유 파악하고 제거 또는 대체를 하면 된다.