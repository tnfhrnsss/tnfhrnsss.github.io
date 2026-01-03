---
layout: post
title: 페이징 시 이전 페이지 데이터가 중복 조회되는 이유와 해결 방법
date: 2026-01-03 15:41:00
last_modified_at : 2026-01-03 15:41:00
parent: Jpa
grand_parent: Msa
nav_exclude: true
tags: [jpa, hibernate]
---


## ⚙️ 현상 요약

- **DB:** Oracle 19c  
- **ORM:** Spring Data JPA  
- **총 데이터:** 1333명  
- **페이징 단위:** 1000명  

정상이라면:
- 1페이지 → 1~1000  
- 2페이지 → 1001~1333  

하지만 실제로는  
👉 **2페이지 결과에 1페이지의 일부 데이터가 포함**되는 현상이 발생

---

## ❌ JPA 문제인가요?

이 현상은:
- MyBatis
- JDBC
- 순수 SQL
- 어떤 ORM / 프레임워크

를 사용하더라도 동일한 조건으로 구현했다면, JPA여부에 상관없이 발생하는 것인데요.
어찌보면 당연한 것이였는데, 
JPA라는 상황에 함몰되 기본을 잊었었습니다 ㅜㅠ ..

JPA는 단지 SQL을 생성해서 DB에 전달할 뿐,  
정렬 순서나 OFFSET 계산에 관여하지 않습니다.

---

## 🧠 원인: 비결정적 ORDER BY

> **ORDER BY 결과가 유일하지 않으면, 행의 순서는 보장되지 않는다**

예를 들어 다음 쿼리를 보겠습니다.

```sql
ORDER BY created_at DESC
```

`created_at` 값이 동일한 행이 여러 개 존재하면,  
그 행들 사이의 순서는 DB 내부 구현에 따라 **매번 달라질 수 있습니다.**

이 상태에서 OFFSET 페이징을 하면:

1. DB가 결과를 정렬하고
2. 앞에서 N개를 버린 뒤 (OFFSET)
3. 다음 M개를 가져오는데

👉 **N번째 경계가 흔들리면서**
- 이전 페이지 데이터가 다시 나오거나
- 일부 데이터가 누락됩니다.

문제가 되었던 것은 created_at에 해당하는 컬럼의 데이터가 char유형의 8자리 일자 데이터로 저장되는 비결정형적
데이터였던 것입니다. (예 : 20260101)

---

## ✅ 해결 방법: 정렬을 “결정적으로” 만들기

정렬 기준에 **고유한 컬럼(PK)** 으로 지정해야합니다.

```sql
ORDER BY created_at DESC, id DESC
```

이렇게 하면:
- 같은 `created_at` 값을 가진 행도
- `id` 기준으로 순서가 확정되고
- OFFSET 경계가 고정됩니다.

사실은 문제가 되었던 비지니스 외에
long타입의 타임스탭프 컬럼을 기준으로 sorting을 하는 로직들이 많은데요.
동시접속이 많은 경우도 충분히 단일컬럼만으로는 문제가 발생할 수 있겠습니다.

---

## 💡 JPA 코드 예시

Spring Data JPA:

```java
PageRequest.of(
    page,
    size,
    Sort.by(
        Sort.Order.desc("createdAt"),
        Sort.Order.desc("id")
    )
);
```

---

## 🧾 검증

hibernate 쿼리 로그레벨을 설정하면 쿼리를 디버깅모드로 확인해볼 수 있습니다.

