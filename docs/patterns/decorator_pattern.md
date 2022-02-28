---
layout: post
title: Decorator pattern
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Home
# parent: Patterns
# has_children: false
nav_order: 6
nav_exclude: true
---

# Decorator pattern

데코레이터 패턴

- 예를 들어서, Set인스턴스를 감싸는(wrap) 래퍼 클래스를 만들고, 이 클래스에 계측 기능을 덧씌우는 것을 말함

```java
static void walk(Set<Dog> dogs) {
	InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
}
```