---
layout: post
title: elastic search response long type to change date format
date: 2023-05-25 08:45:00
last_modified_at : 2023-05-25 08:45:00
parent: Elastic Search
grand_parent: Msa
nav_exclude: true
---

# have to do

- elasticsearch에 long 타입으로 타임스탭프가 저장된 데이터를 _search api를 통해 조회할때, date형태로 변환해서 조회해야했다.
- script를 통해서 변환이 가능하다는 것을 확인!
- lang은 기본 painless를 사용했다.

# error

- 처음엔 이렇게 작성했다

```xml
GET message_idx/_search
{
 "_source": ["createdAt"], 
  "script_fields": {
    "test_script": {
      "script": {
        "source": "doc['createdAt'].value.toString('yyyy-MM-dd HH:mm:ss')"
      }
    }
  } 
}
```

```xml
"type" : "search_phase_execution_exception",
    "reason" : "all shards failed",
    "phase" : "query",
    "grouped" : true,
    "failed_shards" : [
      {
        "shard" : 0,
        "index" : "message_idx",
        "node" : "hhcJgG8dS4WX3FnLATaFEA",
        "reason" : {
          "type" : "script_exception",
          "reason" : "runtime error",
          "script_stack" : [
            "doc['createdAt'].value.toString('yyyy-MM-dd HH:mm:ss')",
            "                      ^---- HERE"
          ],
          "script" : "doc['createdAt'].value.toString('yyyy-MM-dd HH:mm:ss')",
          "lang" : "painless",
          "position" : {
            "offset" : 22,
            "start" : 0,
            "end" : 54
          },
          "caused_by" : {
            "type" : "illegal_argument_exception",
            "reason" : "dynamic method [java.lang.Long, toString/1] not found"
          }
        }
      }
    ]
```

- 원인은 caused_by를 보면 알 수 있다.
- painless에 long타입을 toString할 수 있는 함수가 없다는 것.

# solved

```xml
GET message_idx/_search
{
 "_source": ["createdAt"], 
  "script_fields": {
    "test_script": {
      "script": {
        "source": "SimpleDateFormat sdf = new SimpleDateFormat('yyyy-MM-dd HH:mm:ss'); sdf.setTimeZone(TimeZone.getTimeZone('GMT+09:00')); String output = sdf.format(new Date(doc['createdAt'].value)); return output"
      }
    }
  } 
}
```

- java를 작성한다고 생각하면 될 꺼같다. 여러 구문은 ; 로 이어서 작성 가능하고, 심지어 로컬 변수선언하고 리턴으로 마무리
- 이러한 스크립트는 저장해두고 사용이 가능하다고 하다.

# Reference

[https://velog.io/@soyeon207/ES-8.-Script](https://velog.io/@soyeon207/ES-8.-Script)