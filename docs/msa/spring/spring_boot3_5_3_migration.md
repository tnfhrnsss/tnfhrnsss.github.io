---
layout: post
title: Spring Boot 3.5.3 & Java 17 마이그레이션 개발 (ing…)
date: 2025-09-21 10:42:00
last_modified_at : 2025-09-21 10:42:00
parent: Spring
grand_parent: Msa
nav_exclude: true
---


## 📌 버전 업그레이드 요약

| 항목 | Before | After |
| --- | --- | --- |
| **Java** | 1.8 | 17 |
| **Spring Boot** | 2.7.18 | 3.5.3 |
| **Spring Cloud** | 2021.0.9 | 2025.0.0 |
| **Maven** | 3.6.1 | 3.9.11 |
| **SLF4J** | 1.7.36 | 2.0.17 |

# 작업 배경

- 노션에 내용을 너무 이것저것 대충 모았더니, 정리가 안되어서 chatGpt에게 1차 정리를 요청했습니다..하..
- 그래도 마이그레이션을 하게 된 배경을 정리하면
    - 오래된 숙제였기도 하고
    - 고객사 요청이기도 했습니다.
    - 개인적으로는 제품에 spring ai를 개발해볼수 있는 환경이 된듯하여 기대되고
    - 또 fatjar 속도 개선이 되었다는 블로그를 보니 더더욱 완성품이 기대됩니다 😆

# 범위 및 일정
- 범위는 현재 제품군 전체에 대한 적용입니다.
- 순차적용되고 있기도하고, 다른 스프린트와 병행해서 진행하기 떄문에 시간 산정이 어려운 점이 아쉽긴하네요. 🥲

## 🔧 공통 적용(모든 어플리케이션)

### 1. 네임스페이스 변경

- `javax.*` → `jakarta.*`
    
    (`annotation`, `persistence`, `validation`, `servlet` 전부 변경)
    

### 2. 테스트 코드

- JUnit 5 적용 (Spring Boot 3.x 기본)
- `@MockBean`은 deprecated 경고 → 현행 유지 (추후 리팩토링 고려)
    - 현행 유지하게 된 이유는 [테스트-코드-수행-속도-개선](https://tnfhrnsss.github.io/docs/quality/testcase/optimization_of_test_case/#3-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C-%EC%88%98%ED%96%89-%EC%86%8D%EB%8F%84-%EA%B0%9C%EC%84%A0){:target="_blank"} 와 관련이 있는데요.
    - 성능 이슈로 asciidoc multipage로 하면서 MockBean으로 컨피그 분리했는데, deprecated되어서 warning발생하는 것으로
    - 좀 더 고민이 필요하다.

### 3. Hibernate 6 호환성 대응

- **EmbeddedId** 접근 방식 변경 → `@Query`로 명시적 쿼리 작성
    - 즉, JpoId 클래서에 extends받아서 부모.jpoid.id로 조회하는건 쿼리메소드로 지원하지 않는다.
        
        ```
        // ❌ Hibernate 6에서 작동하지 않음
        List<Entity> findAllByEmbeddedIdParentIdChildId(String childId);
        ```
        
    - **해결:** @Query 어노테이션 사용
        
        ```
        // ✅ Hibernate 6 호환 방식
        @Query("SELECT e FROM Entity e WHERE e.embeddedId.parentId.childId = :childId")
        List<Entity> findAllByChildId(@Param("childId") String childId);
        ```
        
- **@Embeddable + @MappedSuperclass 동시 사용 불가** → 별도 클래스 구조로 변경
    - as-is
        
        ```
        // ❌ Hibernate 6에서 오류 발생
        @Embeddable
        @EqualsAndHashCode(callSuper = true)
        public class EntityId extends BaseId {
            // ...
        }
        
        ```
        
    - **해결:** Embed 구조로 전환
        
        ```
        // ✅ Hibernate 6 호환 방식
        @Embeddable
        @EqualsAndHashCode
        public class EntityId implements JsonSerializable {
            @Embedded
            private BaseId baseId;
        
            @Override
            public String toString() {
                return toJsonWithoutPretty();
            }
        }
        
        ```
        
- Criteria API 사용 시 **경로 명시 필요**
    
    ```prolog
    // 기존
    root.get(FaqAgentSettingJpoId_.FAQ_AGENT_SETTING_ID).get(field).in(values);
    
    // 변경
    root.get(FaqAgentSettingJpoId_.FAQ_AGENT_SETTING_ID)
       .get(FaqAgentSettingJpoId_.PARTITION_ID)
       .get(field)
       .in(values);
    
    ```
    

### 4. CrudRepository.deleteById() 동작 변화

- 2.x: 존재하지 않는 ID → `EmptyResultDataAccessException`
- 3.x: 존재하지 않는 ID → 아무 동작 없이 성공 (200 OK)
- 해결: 커스텀 JpaRepository 구현체로 예외 던지도록 수정

### 5. Validation 동작 변화

- 3.x에서는 `@RequestParam` + `@Range` 등 제약이 자동 적용
- 2.x에서는 `@Validated`를 붙여야만 동작
    
    → API 테스트 시 결과 차이가 발생할 수 있음
    

## 🗂️ 기타 모듈별 마이그레이션 내역 주요 부분 정리

- Gateway 라우팅 구조 WebFlux 기반으로 변경
- HttpMethod, ResponseStatusException API 변경 반영
- Elasticsearch Script API 최신화
- Maven Wrapper 3.9.11로 업그레이드
- `jakarta.servlet.http.HttpServletRequest` 적용
- Java 17에서 `URLEncoder.encode` 표준 변경 반영
- `jib-maven-plugin` 3.x 호환성 확인 및 설정 수정
- LoadBalancerClient 사용 시 경고 확인 (Spring 7.0 변경 사전 안내)
- Feign GET 파라미터 명시 (`@RequestParam`)
- Hibernate Dialect 변경: `MySQLDialect`

## 🛠️ 발생한 이슈 & 해결

### 1. SLF4J 충돌

- `LoggerFactory is not a Logback LoggerContext` 오류 발생
- 원인: slf4j 1.7.36 → 2.0.17 업그레이드 시 구버전 JAR 잔존
- 해결: classpath 정리 + logback 버전 동기화

### 2. AOP JoinPointMatch 오류

```java
Required to bind 2 arguments, but only bound 1 (JoinPointMatch was NOT bound)

```

- 원인: @Order 우선순위 너무 높음
- 해결: `@Order(Ordered.HIGHEST_PRECEDENCE)` → `@Order(1)`

## 3. Validated 유효성 버그 패치?!

- 원래는 validated가 선언되어있지 않으면, `@Range`, `@Min`, `@Max`, `@NotBlank` 같은 Bean Validation를 선언해도 적용이 안되는 코드가 있었다면, 기존에는 오류가 나지 않았었는데요,
- 이번에 작업하면서 그러한 잘못 선언된, validated 어노테이션없이 valid 어노테이션을 사용한 경우는 오류가 나도록 패치가 된것 같습니다.
- 그러면서 코드 내에 버그도 잡게 된...
- `@Validated` 가 없는데, 아래 `@Range` 가 기존 버전에서는 작동하지 않았고, spring 3 에서는 작동한 경우입니다.
    
    ```
    public BtalkRdoListRdo findAll(
       ...
       @Range(min = 1, max = 100) @RequestParam(defaultValue = "10", required = false) int limit,
       ...
    
    ```
    

## 📊 SonarQube 개선 사항

- 소나큐브 버전 : **`SonarQube version: 9.9.1.69595`**
- **Java 16+ 기능 적용**
    - java: S6201 (minor) Pattern Matching for `instanceof`
        - 이건 확실이 라인 수가 줄어드니 좋더군요! ㅋㅋ
        - 기존코드도 동작이 없지만, 컴파일시 타입체크 보장되고
        - 의도가 명확해서 가독성이 좋아지고
        - 캐스팅과 변수선언을 한번에 하기때문에 코드가 간결해집니다.
        - 코드
            - as-is
                
                ```java
                if (obj instanceof Message) {
                    return validateBy(((Message) obj), productType);
                } else if (obj instanceof ConversationMessengerMessage) {
                    return validate(((ConversationMessengerMessage) obj).getConversationChannel(), productType);
                } else if (obj instanceof ConversationMessengerClosedMessage) {
                    return validate(((ConversationMessengerClosedMessage) obj).getConversationChannel(), productType);
                ```
                
            - to-be
                
                ```java
                if (obj instanceof Message message) {
                    return validateBy(message, productType);
                } else if (obj instanceof ConversationMessengerMessage conversationMessengerMessage) {
                    return validate(conversationMessengerMessage.getConversationChannel(), productType);
                } else if (obj instanceof ConversationMessengerClosedMessage conversationMessengerClosedMessage) {
                    return validate(conversationMessengerClosedMessage.getConversationChannel(), productType);
                }
                ```
                
    - **Java:S6204(major) Replace this usage of 'Stream.collect(Collectors.toList())' with 'Stream.toList()'"(major)**
        - `Stream.toList()` 사용 권장
        - 기존에는 Collectors.toList로 변환하던 부분에서 가변, 불변인지 분류해서
        - 가변인 경우는 기존처럼,
        - 불변인 경우는 toList를 사용하는 것으로 변경했습니다.
        - java 10+ 이후 적용가능합니다.
        
        | 구분 | collect(Collectors.toList()) | toList() |
        | --- | --- | --- |
        | **반환 타입** | ArrayList (가변) | List.of() (불변) |
        | **코드 길이** | 길음 | 짧음 |
        | **Java 버전** | Java 8+ | Java 16+ |
        | **성능** | 약간 느림 | 약간 빠름 |
        - 언제 **Collectors.toList()를 사용하나?**
            - 반환된 리스트를 호출자가 수정해야 하는 경우
            - 리스트에 요소를 추가/삭제해야 하는 경우
            - 리스트의 요소들을 수정해야 하는 경우
            - 런타임시에 버튼 추가/삭제
            - 조건부 필터링 후 리스트 수정
            - 사용자 상호작용에 따른 동적 변경
            - 실시간 업데이트가 필요한 경우
            - 버튼 순서 변경
        - 코드
            - as-is
                
                ```java
                Arrays.stream(businessChannelTypes)
                .collect(Collectors.toList());
                ```
                
            - to-be
                
                ```java
                Arrays.stream(businessChannelTypes).toList();
                ```
                
    - java:**S6212 (info)** Declare this local variable with "var" instead.
        - `var` 적용 권장—> 이건 룰 제외로 처리했습니다.
        - java 10부터 도입된 기능으로, 컴파일러가 타입을 자동으로 추론할 수 있기 때문에
        - 타입추론이 가능한 지역변수는 var로 선언해라.라는 의도입니다.
        - 하지만 크게 성능적인 이점이 있거나 그런것도 없고, info레벨인데다가. 형을 알수 없는 단점이 있어서 다른 분들과 얘기한 후 룰 제외로 정리했습니다.
    - java:S6126(major) Replace this String concatenation with Text block.
        - 원인 : 긴 문자열을 + 연산자로 연결하지말고 Java 15부터 도입된 **Text Block("""로 시작하고 끝나는 멀티라인 문자열)**을 사용
        - 코드
            - as-is
            
            ```java
            "{\n  \"applicationId\" : \"btalk\"
            ",\n  \"messageId\" : 2034186899619840
            "ne\",\n    \"message\" : null\n  }\n}",
            ```
            
            - to-be
            
            ```java
            """
            	{
            	  "applicationId" : "btalk",
            	  "messageId" : 2034186899619840,
            	  "channelUrl" : "d0f31e10-a9e1-4f6f-ba99-108c73ca2fdd",
            	  "type" : "text",
            	  ...
            	  }
            	}
            """,
            ```
            
    - java:S1874**(minor)** **Remove this use of "defaultString"; it is deprecated.**
        - 원인 : deprecated됨
    - java:S6208(info) Merge the previous cases into this one using comma-separated label.
        - 원인 : switch 문에서 여러 case들이 같은 동작을 수행하고 있는데, 이들을 쉼표로 구분하여 하나의 case로 병합하라는 것
        - 코드
            
            ```java
            // Java 10 이전 (기존 방식)
            switch (value) {
                case 1:
                case 2:
                case 3:
                    System.out.println("1, 2, or 3");
                    break;
            }
            
            // Java 10 이후 (새로운 방식)
            switch (value) {
                case 1, 2, 3:
                    System.out.println("1, 2, or 3");
                    break;
            }
            ```
            
        - 또는 enhanced switch도 가능
            
            ```java
            switch (warningConversationMessage.getWarningMessageType()) {
                  case NOT_ALLOWED_MESSAGE -> throw new NotAllowedMessageException(KakaoSystemMessageType.S5.name());
                  case NO_OPERATING_TIME, NOT_ALLOWED_REQUEST ->
                          throw new GeneralMessagingException(KakaoSystemMessageType.S2.name());
                  case NOT_AVAILABLE_MESSAGE -> throw new GeneralMessagingException(KakaoSystemMessageType.S1.name());
                  default -> {
                  }
              }
            ```
            
    

# ✅ 작업 후기 및 남은 작업

- 작년에 [spring_boot3_upgrade/](https://tnfhrnsss.github.io/docs/msa/spring/spring_boot3_upgrade/){:target="_blank"} 에서 미리 준비하면서 정리했던 것이 그나마 좀 도움이 되었던 것 같습니다.
- ui 및 api 테스트가 진행중이지만, 제가 디버깅하면서 검증해야할 비기능 요구사항도 있어서, 조금씩 해야할 것 같습니다.
    - LoadBalancer 테스트
    - Spring Cloud OpenFeign 4.x 설정 (TTL, retry, logging, HttpClient5 적용)
- 이번 마이그레이션 스프린트에 참여하지 않은 다른 개발자에게도 전파해야할 것 같아서 잘 정리해나가볼 생각입니다.