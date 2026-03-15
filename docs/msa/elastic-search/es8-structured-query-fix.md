---
layout: post
title: Elasticsearch 8.x 업그레이드 후 elasticsearch-head Structured Query 필드가 안 보이는 문제 해결
date: 2026-03-09 21:24:00
last_modified_at : 2026-03-09 21:24:00
parent: Elastic Search
grand_parent: Msa
nav_exclude: true
tags: [elasticsearch, backup]
description: "Elasticsearch 7.x → 8.x 업그레이드 후 elasticsearch-head Structured Query 탭에 필드가 사라지는 원인(hit._type 제거)과 한 줄 수정으로 해결하는 방법"
---


## 증상

Spring Boot 버전업과 함께 Elasticsearch를 **7.x → 8.x**로 업그레이드한 뒤,
`Multi Elasticsearch Head` Chrome 확장의 **Structured Query** 탭에서 검색 시
기존에는 `_class`, `id`, `name` 등 매핑된 필드들이 헤더와 함께 표시되던 것이
`_index`, `_type`, `_id`, `_score` **4개 컬럼만 표시**되고 실제 데이터 필드가 모두 사라졌다.

---

## 원인 분석

### ES 7.x vs ES 8.x 응답 차이

ES 7.x 검색 결과:
```json
{
  "hits": {
    "hits": [
      {
        "_index": "my-index",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.0,
        "_source": { "id": 1, "name": "홍길동" }
      }
    ]
  }
}
```

ES 8.x 검색 결과:
```json
{
  "hits": {
    "hits": [
      {
        "_index": "my-index",
        "_id": "1",
        "_score": 1.0,
        "_source": { "id": 1, "name": "홍길동" }
      }
    ]
  }
}
```

**ES 8.x부터 `_type` 필드가 완전히 제거**되었다.
`hit._type`은 `undefined`가 된다.

---

### 코드 내부에서 무슨 일이 벌어지는가

`app.js`의 `data.QueryDataSourceInterface._getData` 함수에서
검색 결과의 각 필드를 메타데이터 경로(path)와 매핑하여 컬럼으로 추가한다.

```js
// app.js - 문제가 된 코드
var row = (function (path, spec, row) {
    for (var prop in spec) {
        // ...
        var dpath = path.concat(prop).join(".");
        if (metadata.paths[dpath]) {           // ← 여기서 항상 false
            columns.push(field_name);
            row[field_name] = ...;
        }
    }
    return row;
})([hit._index, hit._type], hit._source, {}); // ← hit._type이 undefined
```

메타데이터(`data.MetaData`)는 이미 ES 8+ 호환 코드가 적용되어
`_doc`을 기본 타입명으로 사용해 경로를 저장한다:

```
metadata.paths["my-index._doc.id"]     // ✅ 저장된 경로
metadata.paths["my-index.undefined.id"] // ❌ 실제 조회되는 경로
```

두 경로가 불일치하여 `metadata.paths[dpath]`가 항상 `false`를 반환하고,
결과적으로 어떤 필드도 컬럼으로 추가되지 않는다.

---

## 해결 방법

`hit._type`이 `undefined`일 경우 `"_doc"`으로 폴백하도록 수정.

### 변경 파일: `elasticsearch-head/app.js`

```diff
- })([hit._index, hit._type], hit._source, {});
+ })([hit._index, hit._type || "_doc"], hit._source, {});
```

**수정 위치**: `data.QueryDataSourceInterface._getData` 함수 내부
(검색 키워드: `hit._source, {}` )

---

## 왜 이 한 줄로 해결되는가

| 구분 | ES 7.x | ES 8.x (수정 전) | ES 8.x (수정 후) |
|---|---|---|---|
| `hit._type` 값 | `"_doc"` | `undefined` | `undefined` |
| 경로 prefix | `my-index._doc` | `my-index.undefined` | `my-index._doc` |
| 메타데이터 경로 | `my-index._doc.id` | `my-index._doc.id` | `my-index._doc.id` |
| 경로 일치 여부 | ✅ | ❌ | ✅ |
| 필드 컬럼 표시 | ✅ | ❌ | ✅ |

ES 8+ 호환 메타데이터 파싱 코드는 이미 `_doc`을 기본 타입으로 사용하고 있어서,
검색 결과 경로 생성 쪽만 맞춰주면 된다.

---

## 적용 환경

- **Elasticsearch**: 7.x → 8.x (확인: 8.18.3)
- **확장**: [Multi Elasticsearch Head](https://github.com/vorapoap/elasticsearch-head-chrome)

---

## 참고

- [Elasticsearch 8.0 Breaking Changes - Removal of mapping types](https://www.elastic.co/guide/en/elasticsearch/reference/8.0/breaking-changes-8.0.html#breaking_80_rest_api_changes)
- ES 8.x에서 타입(type) 개념 자체가 제거되었으며, 기존 `_doc`이 기본값이었던 것도 응답에서 완전히 생략된다.
