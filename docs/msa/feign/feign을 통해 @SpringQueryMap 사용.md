---
layout: default
title: feign을 통해 @SpringQueryMap 사용
parent: Feign
grand_parent: Msa
nav_order: 1
---

# feign을 통해 @SpringQueryMap 사용

![Untitled](/assets/images/msa/feign.png)

### ::요구사항

고객의 과거 상담 이력을 조회해서 다른 서비스로 전달하는 로직

### **:: History API**

```java
@RestController
@RequestMapping("histories")
public class HistoryQueryResource {
    private final HistoryQueryService historyQueryService;

    @GetMapping
    public HistoryList findAll(
        @SpringQueryMap HistoryQuery historyQuery
    ) {
        return historyQueryService.findAll(historyQuery);
    }
}
```

### ::Thirdparty

```java
@FeignClient(
    contextId = "HistoryQueryClient",
    name = "api",
    configuration = {FeignConfiguration.class, FeignLoggerLevelConfiguration.class},
    primary = false
)
public interface HistoryQueryClient{
    @GetMapping("histories")
    HistoryList findAll(@SpringQueryMap HistoryQuery historyQuery);
}
```

### :: 문제

- 컨트롤러가 Mapping파라미터로 선언한 HistoryQuery에 ArrayList, Map타입이 아니면 맵핑이 되지 않는 오류발생

```java
@Builder
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@AllArgsConstructor
@Getter
@ToString
public class HistoryQuery implements JsonSerializable {
    @Builder.Default
    private int offset = 0;   // --> API서버로 맵핑되지 않음

    @Builder.Default
    private int limit = 1;    // --> API서버로 맵핑되지 않음

    @Builder.Default
    private IdList channelIds = IdList.empty(); // --> 맵핑ok

    @Builder.Default
    private IdList status = IdList.empty();  // --> 맵핑ok

    @Builder.Default
    private String customerId = StringUtils.EMPTY;  // --> API서버로 맵핑되지 않음
}
```

### :: 문제

1. 호출 URL에는 문제가 없지만, History API 서버에는 offset, limit, customerId가 찍히지 않음
    
    ```
    -> GET http://api/histories?offset=0&status=closed&limit=1
    &customerId=tester&channelIds=kakao
    ```
    
2. feign doc

[Spring Cloud OpenFeign](https://cloud.spring.io/spring-cloud-openfeign/reference/html/#feign-querymap-support)

doc을 보면, POJO, Map 모두 지원한다고 되어있지만, 테스트해보니 리스트나 맵이 아니면 안됨

### :: 문제해결

1. Param.Expander를 사용하는 방법 —> 받는 쪽이 array, map이 아니면 이것도 안됨

[Feign Client does not resolve Query parameter | Newbedev](https://newbedev.com/feign-client-does-not-resolve-query-parameter)

1. API 스펙을 변경하는 방법....