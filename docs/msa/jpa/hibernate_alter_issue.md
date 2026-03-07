---
layout: post
title: Hibernate6 '@Column(length) 설정 시 매 시작마다 ALTER TABLE이 실행되는 버그
date: 2026-03-07 12:00:00
last_modified_at : 2026-03-07 12:00:00
parent: Jpa
grand_parent: Msa
nav_exclude: true
tags: [jpa, hibernate]
---


## 현상

Spring Boot 3.x + Hibernate 6.x + MySQL 환경에서 `ddl-auto=update`로 설정 시,
JPA Entity 클래스를 전혀 수정하지 않았는데도 **서비스가 시작될 때마다** 아래와 같은 ALTER TABLE 로그가 찍힌다.

```sql
alter table thirdparty_conversation
   modify column c_messenger_channel_type enum ('ETC','FAQ','KAKAO','LINE','NAVER')

alter table thirdparty_conversation
   modify column c_product_type enum ('BTALK','CTALK')
```


---

## 원인

### Entity 코드

```java
@Enumerated(EnumType.STRING)
@Column(length = 20)
private MessengerChannelType messengerChannelType;

@Enumerated(EnumType.STRING)
@Column(length = 20)
private ProductType productType;
```

얼핏 보면 문제없어 보이지만, **Hibernate 6에서 동작이 바뀌었다.**

### Hibernate 5 vs Hibernate 6 enum 매핑 변화

| 버전 | `@Enumerated(EnumType.STRING)` 기본 매핑 |
|------|------------------------------------------|
| Hibernate 5 | VARCHAR |
| Hibernate 6 | MySQL native enum 타입 |

Hibernate 6부터 MySQL/MariaDB에서 `@Enumerated(EnumType.STRING)`은 기본적으로 DB의 native enum 타입으로 매핑된다.

### 문제의 핵심: `@Column(length=20)`이 코드 경로를 바꾼다

```
[length 있을 때]
  Hibernate: "length 속성이 있으니 VARCHAR 계열 비교 로직 진입"
  DB 실제 타입: enum('BTALK','CTALK')
  비교 결과: "다르다" → ALTER TABLE 생성

[length 없을 때]
  Hibernate: "enum 타입 비교 로직 진입"
  DB 실제 타입: enum('BTALK','CTALK')
  비교 결과: "같다" → ALTER 없음
```

`@Column(length=20)`의 `length` 속성이 Hibernate 6의 스키마 비교 코드 경로에 간섭하여,
**DB와 실제로 동일한 스키마임에도 "다르다"고 잘못 판단**하게 만든다.

### 실행되는 ALTER는 실제로 아무 변화가 없다

생성되는 ALTER 구문을 보면:

```sql
modify column c_messenger_channel_type enum ('ETC','FAQ','KAKAO','LINE','NAVER')
```

DB에 이미 동일한 enum 정의가 있음에도 그대로 다시 쓰는 **no-op ALTER**다.
데이터나 스키마에 실질적인 영향은 없지만, 매 시작마다 불필요하게 실행된다.

---

## 해결 방법

### 방법 1: `@Column(length)` 제거 — DB native enum 유지

```java
// 변경 전
@Enumerated(EnumType.STRING)
@Column(length = 20)
private MessengerChannelType messengerChannelType;

// 변경 후
@Enumerated(EnumType.STRING)
private MessengerChannelType messengerChannelType;
```

- ALTER 즉시 사라짐
- DB 마이그레이션 불필요
- 단, DB에 native enum이 유지되므로 enum 값 추가 시 DDL 마이그레이션 필요

### 방법 2: VARCHAR로 명시 — `@JdbcTypeCode` 사용

```java
import org.hibernate.annotations.JdbcTypeCode;
import org.hibernate.type.SqlTypes;

@Enumerated(EnumType.STRING)
@JdbcTypeCode(SqlTypes.VARCHAR)
@Column(length = 20)
private MessengerChannelType messengerChannelType;
```

- Hibernate에게 "이 컬럼은 VARCHAR"라고 명시적으로 지정
- `@Column(length=20)` 유지 가능
- DB 컬럼이 이미 native enum이라면 VARCHAR로 한 번 마이그레이션 필요

```sql
ALTER TABLE thirdparty_conversation
  MODIFY COLUMN c_messenger_channel_type VARCHAR(20);
```

### 방법 3: columnDefinition 명시

```java
@Enumerated(EnumType.STRING)
@Column(columnDefinition = "enum('ETC','FAQ','KAKAO','LINE','NAVER')")
private MessengerChannelType messengerChannelType;
```

- enum 정의를 코드에 하드코딩하는 방식
- enum 값 추가 시 코드와 DDL 모두 수정 필요하여 비권장

---

## 어떤 방법을 선택해야 하나

**enum 값이 고정된 경우** (예: Y/N, 상태코드 등)
→ 방법 1 또는 방법 3으로 native enum 유지

**enum 값이 늘어날 수 있는 경우** (예: 새 플러그인/채널 타입 추가 등)
→ **방법 2 (VARCHAR) 권장**

native enum은 값이 추가될 때마다 DDL 마이그레이션이 필요하다.
반면 VARCHAR는 코드에서 새 enum 값을 추가하는 것만으로 충분하고,
DB 레벨의 무결성은 어차피 JPA/애플리케이션 레이어에서 보장된다.
실무에서도 `@Enumerated(EnumType.STRING)` + VARCHAR가 사실상 표준이다.

---

## 환경

- Spring Boot 3.x
- Hibernate 6.x
- MySQL / MariaDB
- `spring.jpa.hibernate.ddl-auto=update`

---

## 참고

- [Hibernate ORM Issue Tracker](https://hibernate.atlassian.net/browse/HHH)
- Hibernate 6 Migration Guide: Enum mapping changes
