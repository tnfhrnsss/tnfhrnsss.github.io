---
layout: post
title: elasticsearch field add(1)
date: 2021-09-09 13:07:45
last_modified_at : 2022-03-19 12:38:00
parent: Elastic Search
grand_parent: Msa
nav_exclude: true
tags: [elasticsearch]
---

<aside>
๐ก ์ง๊ธ๊น์ง ์ฐพ์๋ณธ ๋ฐฉ์ ์ค 2๊ฐ์ง๋ฅผ ์ ๋ฆฌ

</aside>

[๊ณตํต] 

๊ธฐ์กด ๋ฐ์ดํฐ ๋ฐฑ์์ ์งํํฉ๋๋ค.

์ค๋์ท ๋ฐฑ์ ๋ฐฉ์์ ["์ฌ๊ธฐ"](elasticsearch%20%E1%84%89%E1%85%B3%E1%84%82%E1%85%A2%E1%86%B8%E1%84%89%E1%85%A3%E1%86%BA%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%87%E1%85%A9%E1%86%A8%E1%84%8B%E1%85%AF%E1%86%AB%20%E1%84%89%E1%85%A1%E1%86%A8%E1%84%8C%E1%85%A6%20ab7073fe4fe247b19f91220552094a42.md) ์ฐธ๊ณ ํ์ธ์.

# [๊ธฐ์กด ๋ฐ์ดํฐ์ ์ถ๊ฐํ๋ ๋ฐฉ์]

<aside>
๐ก *๋ฐ์ดํฐ๊ฐ ์ ๊ณ , ์๋ฒ์ ๋ฌธ์ ๊ฐ ์์ ๊ฒฝ์ฐ์๋ง ์ฌ์ฉํฉ๋๋ค.  (๋ก์ปฌ์ ๊ฒฝ์ฐ์๋ง ์งํํ๋๋ก ํฉ๋๋ค.)*

</aside>

### 1. ํ์ฌ ์ ์ฉ๋ ๋งตํ ์ ๋ณด๋ฅผ ์กฐํํ๋ค.

```json
GET attic_ticket/_mapping
```

### 2. ์์์ ์กฐํํ ๋งตํ ์ ๋ณด๋ฅผ ์ฐธ๊ณ ํด์, depth์ ๋ง๊ฒ ์ปฌ๋ผ์ ์ถ๊ฐํ๋ค.

```json
PUT attic_ticket/_mappings
{
    "properties": {
      "customer" : {
        "type":"nested",
        "properties": {
          "messengerId": {    // customer์ messengerId ์ปฌ๋ผ ์ถ๊ฐ
            "type": "text"
          }
        }
      }
  }
}
```

![../img/els2-1.png](../img/els2-1.png)

### 3. ์ ์ถ๊ฐ๋์๋์ง ํ์ธํ๋ค

```json
GET attic_ticket/_mapping
```

### 4. ๊ธฐ์กด ๋ฐ์ดํฐ์ ์ถ๊ฐํ ํ๋๋ฅผ ์๋ฐ์ดํธ ํ๋ค.( multiple update )

```json
POST attic_ticket/_update_by_query
{
  "script": "ctx._source.customer.messengerId = ''"
}
```

# ** (ํ๋๊ฐ ๋ฐฐ์ด์ธ ๊ฒฝ์ฐ) **

### 5-1. ๊ธฐ์กด ๋ฐฐ์ด ์ญ์ 

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

### 5-2. ๋ฐฐ์ด ์ด๊ธฐํ

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

### 5-3. ๋ฐฐ์ด์ ๊ฐ ์ถ๊ฐ

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

### 5.  ๊ฒ์ฆ

1. ์ผ๋ถ ํฐ์ผ์๋ง ์ถ๊ฐํ ํ๋์ ๊ฐ์ ์๋ฐ์ดํธํด๋ณธ๋ค.

```json
POST attic_ticket/_update_by_query
{
  "script": {
    "source":"ctx._source.customer.messengerId = 'honggildong'"
  },
  "query":{     // ์ผ๋ถ ticket์๋ง ๊ฐ์ ์๋ฐ์ดํธํ๊ณ 
    "term":{
      "ticketId":"63af9a77-845b-4ca5-b692-0d3e5cc7d398"
    }
  }
}
```

 2. ์ถ๊ฐํ ๊ฐ์ผ๋ก ๊ฒ์ํด๋ณธ๋ค.

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

์ฐธ๊ณ ) [https://esbook.kimjmin.net/04-data/4.3-_bulk](https://esbook.kimjmin.net/04-data/4.3-_bulk)