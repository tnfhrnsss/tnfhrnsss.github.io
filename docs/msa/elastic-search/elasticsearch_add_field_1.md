---
layout: post
title: elasticsearch field add(1)
date: 2021-09-09 13:07:45
last_modified_at : 2022-03-19 12:38:00
parent: Msa
# parent: Elastic Search
# grand_parent: Msa
nav_exclude: true
tags: [elasticsearch]
---

<aside>
💡 지금까지 찾아본 방식 중 2가지를 정리

</aside>

[공통] 

기존 데이터 백업을 진행합니다.

스냅샷 백업 방식은 ["여기"](elasticsearch%20%E1%84%89%E1%85%B3%E1%84%82%E1%85%A2%E1%86%B8%E1%84%89%E1%85%A3%E1%86%BA%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%87%E1%85%A9%E1%86%A8%E1%84%8B%E1%85%AF%E1%86%AB%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%20ab7073fe4fe247b19f91220552094a42.md) 참고하세요.

# [기존 데이터에 추가하는 방식]

<aside>
💡 *데이터가 적고, 서버에 문제가 없을 경우에만 사용합니다.  (로컬의 경우에만 진행하도록 합니다.)*

</aside>

### 1. 현재 적용된 맵핑 정보를 조회한다.

```json
GET attic_ticket/_mapping
```

### 2. 위에서 조회한 맵핑 정보를 참고해서, depth에 맞게 컬럼을 추가한다.

```json
PUT attic_ticket/_mappings
{
    "properties": {
      "customer" : {
        "type":"nested",
        "properties": {
          "messengerId": {    // customer에 messengerId 컬럼 추가
            "type": "text"
          }
        }
      }
  }
}
```

![../img/els2-1.png](../img/els2-1.png)

### 3. 잘 추가되었는지 확인한다

```json
GET attic_ticket/_mapping
```

### 4. 기존 데이터에 추가한 필드를 업데이트 한다.( multiple update )

```json
POST attic_ticket/_update_by_query
{
  "script": "ctx._source.customer.messengerId = ''"
}
```

# ** (필드가 배열인 경우) **

### 5-1. 기존 배열 삭제

```json
POST attic_message/_update_by_query
{
  "script": {
    "source": "ctx._source.remove('metaarray')"
  },
  "query":{
    "bool": {
      "should": [
        {
          "terms": {
            "channelUrl": [
              "92095428-d647-466c-a054-e1e984dd48b1"
            ]
          }
        }
      ]
    }
  }
}
```

### 5-2. 배열 초기화

```json
POST attic_message/_update_by_query
{
  "script": "ctx._source.metaarray = []",
  "query":{
    "bool": {
      "should": [
        {
          "terms": {
            "channelUrl": [
              "92095428-d647-466c-a054-e1e984dd48b1"
            ]
          }
        }
      ]
    }
  }
}
```

### 5-3. 배열에 값 추가

```json
POST attic_message/_update_by_query
{
  "script": {
    "source":"ctx._source.metaarray.add(params)",
    "params": {
      "key": "templateMessage",
      "value":[
                "{~~~~~~~~~~}"
              ]
    }
  },
  "query":{
    "bool": {
      "should": [
        {
          "terms": {
            "channelUrl": [
              "92095428-d647-466c-a054-e1e984dd48b1"
            ]
          }
        }
      ]
    }
  }
}
```

### 5.  검증

1. 일부 티켓에만 추가한 필드에 값을 업데이트해본다.

```json
POST attic_ticket/_update_by_query
{
  "script": {
    "source":"ctx._source.customer.messengerId = 'honggildong'"
  },
  "query":{     // 일부 ticket에만 값을 업데이트하고
    "term":{
      "ticketId":"63af9a77-845b-4ca5-b692-0d3e5cc7d398"
    }
  }
}
```

 2. 추가한 값으로 검색해본다.

```json
GET attic_ticket/_search
{
    "query" : {
      "bool": {
        "should":[{
          "terms":{
            "ticketId":["63af9a77-845b-4ca5-b692-0d3e5cc7d398"],"boost":1.0
          }
        }]
      }
    }
}
```

참고) [https://esbook.kimjmin.net/04-data/4.3-_bulk](https://esbook.kimjmin.net/04-data/4.3-_bulk)