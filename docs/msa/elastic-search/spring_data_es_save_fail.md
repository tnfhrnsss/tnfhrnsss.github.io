---
layout: post
title: Spring Data Elasticsearch에서 custom collection mapping problem
date: 2025-05-09 19:56:00
last_modified_at : 2025-05-09 19:56:00
parent: Elastic Search
grand_parent: Msa
nav_exclude: true
---

# Error

- elasticsearch에 데이터가 생성되지 않음.
- referrers값을 전송하지 않으면 생성됨.
- 에러로그는 없다. 그야말로 조용한 실패다.

# Problem

- 문제의 Test 도큐먼트 모델 클래스다. **referrers**를 이번 스프린트에 신규 추가했다.
- IdList는 ArrayList를 상속받은 커스텀 클래스이다.

```java
@Document(
    indexName = "test",
    writeTypeHint = WriteTypeHint.FALSE
)
@Getter
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@AllArgsConstructor
@EqualsAndHashCode
public class TestDoc {
    @Id
    @Field(type = FieldType.Keyword)
    private String id;

    @Field(type = FieldType.Object, index = false)
    private IdList referrers = IdList.empty();
  
}

// IdList
public class IdList extends ArrayList<String> implements JsonSerializable {
    public IdList(Collection<String> c) {
        super(c);
    }
}
```

# Solved

## 방법1.  커스텀 클래스 IdList를 List<String>으로 변경하고 Keyword타입으로 지정

```java
@Field(type = FieldType.Keyword)
private List<String> referrers;
```

## 방법2. Nested타입으로 선언

- nested타입으로 선언하고 각 내부 필드에
- 예시
    
    ```java
    @Field(type = FieldType.Nested)
    private List<IdObject> referrers;
    
    public static class IdObject {
      @Field(type = FieldType.Keyword)
      private String id;
    }
    ```
    

# Cause

## 1. IdList 의 필드타입을 FieldType.Object로 선언한 부분 문제

- FieldType.Object의 동작방식
    - es는 객체 타입 필드를 내부적으로 flatten해서 저장시키고 있는데, 예를 들면 List<Test>는 해당 Test 클래스의 변수를 property로 해서 test.변수1, test.변수2 형태로 저장된다.
    - 근데 문제는 IdList가 ArrayList를 상속 구현한 클래스여도 es에서 커스텀 컬렉션 타입을 인식못하는 문제가 있다.
    - 그리고 FieldType.Object은 일반 Pojo를 맵핑하고 있고, 해당 Pojo클래스 멤버변수들에도 FieldType 선언이 되어있어야한다. (**명시적 맵핑**)
- 참고로 List<String>을 FieldType.Keyword타입으로 선언하면, es는 Object처럼 분할하지 않고 값으로 저장시킨다.
    - 이 경우 검색/집계에 더 용이해짐

# Debugging

- 해당 인덱스의 맵핑을 확인해서, 추가한 필드의 맵핑 타입을 체크한다.
    
    ```java
    GET /{인덱스명}/_mapping
    ```
    
- 로그레벨 변경
    
    ```java
    logging.level.org.springframework.data.elasticsearch=DEBUG
    ```