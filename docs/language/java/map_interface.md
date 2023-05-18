---
layout: post
title: Map 인터페이스 정리
date: 2023-05-19 08:23:00
last_modified_at : 2023-05-19 08:23:00
parent: Java
grand_parent: Language
has_children: false
nav_exclude: true
---

# Map이란?

- Map은 key-value 쌍으로 데이터를 저장하는 Java의 인터페이스입니다.
- Map은 중복되지 않는 키(Key)와 해당 키에 연결된 값(Value)으로 구성됩니다.

# 구현체

1. HashMap과 Hashtable의 차이점은 무엇인가요?
    - HashMap은 동기화되지 않으며, Hashtable은 동기화된 버전입니다.
    - HashMap은 Null 값을 허용하며, Hashtable은 허용하지 않습니다.
    - HashMap은 비동기 환경에서 사용하기에 적합하며, Hashtable은 멀티스레드 환경에서 사용하기에 적합합니다.
    
2. TreeMap과 HashMap의 차이점은 무엇인가요?
    - TreeMap은 정렬된 순서로 키(Key)를 유지하며 데이터를 저장합니다.
    - HashMap은 순서를 보장하지 않고, 해시 알고리즘을 사용하여 데이터를 저장합니다.
    
3. HashMap의 동작 방식은 어떻게 되나요?
    - HashMap은 해시 테이블을 사용하여 데이터를 저장합니다.
    - 데이터를 저장할 때 키(Key)를 해시 함수에 적용하여 해시 값을 계산하고, 해당 해시 값에 대응하는 버킷에 데이터를 저장합니다.
    - 데이터를 조회할 때도 동일한 해시 함수를 적용하여 해당 버킷을 찾고, 버킷 내에서 일치하는 키를 찾아 값을 반환합니다.
    
4. HashMap에서 동일한 키를 사용하면 어떻게 되나요?
    - HashMap은 중복된 키를 허용하지 않습니다.
    - 동일한 키로 값을 저장하면, 기존 값은 덮어쓰여집니다.
    
5. HashMap에서 데이터를 삭제하는 방법은 무엇인가요?
    - remove() 메서드를 사용하여 특정 키에 해당하는 데이터를 제거할 수 있습니다.