---
layout: post
title: Error deserializing co.elastic.clients.elasticsearch._types.analysis.Analyzer Property 'type' not found
description: Error deserializing co.elastic.clients.elasticsearch._types.analysis.Analyzer Property 'type' not found
date: 2024-04-06 20:57:00
last_modified_at : 2024-04-06 20:57:00
parent: Elastic Search
grand_parent: Msa
nav_exclude: true
---


# Error

```java
Caused by: org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'sectionElasticRepository' defined in elastic.repository.SectionElasticRepository defined in @EnableElasticsearchRepositories declared on ElasticConfig: 
Invocation of init method failed; 
nested exception is org.springframework.beans.BeanInstantiationException: 
Failed to instantiate [org.springframework.data.elasticsearch.repository.support.SimpleElasticsearchRepository]: Constructor threw exception; nested exception is co.elastic.clients.json.JsonpMappingException: 
Error deserializing co.elastic.clients.elasticsearch._types.analysis.Analyzer: 
Property 'type' not found (JSON path: index.analysis.analyzer.autocomplete_search) (line no=1, column no=150, offset=-1)
```

# Problem

- Elasticsearch 7.15 버전 부터 HLRC (High Level Rest Client)가 deprecated 되어서, Java API Client 로 마이그레이션이 작업을 하던 중에 발생했다.
- spring boot application을 start할 수 없었다.

# Cause

- 에러로그를 자세히 봤어야했는데... 빈 등록 오류라고 판단하고 엉뚱한 작업을 진행했었다.
- 결론은 analyzer의 type을 맵핑할 수 없다는 것
- [https://www.skyer9.pe.kr/wordpress/?p=7311](https://www.skyer9.pe.kr/wordpress/?p=7311){:target="_blank"} 참고
- 즉, type을 명시해줘야한다.
    
    ![../img/elasticsearch_analyzer_error.png](../img/elasticsearch_analyzer_error.png)
    
    ```java
    "autocomplete_search": {
        "type": "custom", --> 이게 누락된 것
        "tokenizer": "standard",
        "filter": [
          "lowercase",
          "word_delimiter"
        ]
      }
    ```
    
- 결국에 type정의가 되어있지 않고, 그로 인해 autocomplete analyzer가 생성안되어서 빈까지 등록안된 것.

# Solved
- autocomplete analyzer에 type 추가
    
    

