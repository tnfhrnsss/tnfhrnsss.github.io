---
layout: post
title: Migration from JUnit 4 to JUnit 5
date: 2024-02-27 22:51:00
last_modified_at : 2024-02-27 22:51:00
parent: TestCase
grand_parent: Quality Practice
nav_exclude: true
---

# background

- spring framework 5.3.31
- junit 4.13.2
- depedency
    - junit4코드로 작성해도 junit5엔진으로 테스트할 수 있도록 junit-vintage-engine이 선언된 상태
    
    ```xml
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
      </dependency>
      <dependency>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
        <exclusions>
          <exclusion>
            <artifactId>hamcrest-core</artifactId>
            <groupId>org.hamcrest</groupId>
          </exclusion>
        </exclusions>
      </dependency>
    </dependencies>
    ```
    

# goal

- junit4 → junit5로 전환

# step1.

- junit-vintage-engine 의존성을 제거하거나 exclusion 처리한다.

```xml
 <dependency>
    ...
    <exclusions>
        <exclusion>
            <groupId>org.junit.vintage</groupId>
            <artifactId>junit-vintage-engine</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

# step2.

- junit4 코드를 junit5로 변경한다.

## 주요 변경

- junit-vintage-engine 의존성을 제거하지 않았다면, 변경하려는 어노테이션에 2개의 API가 나올것이다.
    - → **junit-jupiter-api**를 선택하도록 한다.

![movejunit4to5_1](../img/movejunit4to5_1.png)

### 1. `@RunWith(MockitoJUnitRunner.class)` →  `@ExtendWith(MockitoExtension.class)`

### 2. @Test 변경 `import org.junit.Test` → `import org.junit.jupiter.api.Test`

### 3. expected = ~Exception.class → assertThrows

- as-is
    
    ```java
    @Test(expected = GeneralMessagingException.class)
    public void reopen() {
        receptionClientServiceImpl.register(ReceptionSdo.sample());
    }
    ```
    
- to-be
    
    ```java
    @Test
    public void reopen() {
        Executable executable = () -> kakaoUserSelector.findUsers(KakaoConversationMessage.sample());
        assertThrows(NoSuchElementException.class, executable);
    }
    ```
    

### 4. `import org.junit.Assert`→`import org.junit.jupiter.api.Assertions`

### 5. @Spy 변경

- as-is
    
    ```java
    @Spy
    private List<KakaoWriteMessageService> writeMessageKindHandlers = mock(ArrayList.class, withSettings().useConstructor().defaultAnswer(CALLS_REAL_METHODS));
    ```
    
- to-be
    
    ```java
    @Spy
    private List<KakaoWriteMessageService> writeMessageKindHandlers = new ArrayList<>();
    ```
    

### 6. @Before, @After 등 테스트 라이프사이클 어노테이션 →  **org.junit.jupiter.api 패키지 하위로 이동**

- @Before → @BeforeEach or @BeforeAll 와 같이 **`@BeforeAll`**, **`@AfterAll`**, **`@BeforeEach`**, **`@AfterEach`** 로 변경

### 7. @Ignore → `@Disabled`

### 8. JUnitRestDocumentation와 @Rule

- JUnitRestDocumentation 대신 **RestDocumentationExtension** 사용
- @Rule 제거
    
    ```java
    @ExtendWith(RestDocumentationExtension.class)
    public abstract class AbstractResourceTest {
        protected MockMvc mockMvc;
        protected RestDocumentationResultHandler document;
    
        @BeforeEach
        public void setUp(WebApplicationContext context,
                          RestDocumentationContextProvider restDocumentation) {
            this.document = MockMvcRestDocumentation.document("{ClassName}/{methodName}", preprocessRequest(prettyPrint()), preprocessResponse(prettyPrint()));
            this.mockMvc = MockMvcBuilders.webAppContextSetup(context).apply(documentationConfiguration(restDocumentation)).alwaysDo(this.document).build();
        }
    }
    ```
    
- RestDocumentationContextProvider vs ManualRestDocumentation
    - RestDocumentationContextProvider는
        - **`MockMvcRestDocumentation`** 또는 **`WebTestClientRestDocumentation`**을 사용할 때, 자동으로 생성되는 컨텍스트를 사용하여 문서화를 한다.
    - ManualRestDocumentation는 수동으로 문서화를 제어하고자 할때 사용한다.

### 9. `@RunWith(SpringRunner.class)` → `@ExtendWith(SpringExtension.class)`

### 10. @RunWith(Suite.class)

- as-is
    
    ```java
    @RunWith(Suite.class)
    @Suite.SuiteClasses({
    })
    ```
    
- to-be
    - junit-platform-suite-api dependency 추가
    - @Suite로 변경
        
        ```xml
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-suite-api</artifactId>
            <version>1.8.2</version>
            <scope>test</scope>
        </dependency>
        ```
        
        ```java
        @Suite
        @SelectClasses({
        })
        ```
        

### 11.  Remove this ‘public’ modifier.

- 테스트 클래스와 메서드에서 public 접근제어자를 제거한다.
- 아래 블로그에서 junit의 발전과 public 접근제어에 대해 읽어보자
    - [https://mangkyu.tistory.com/280](https://mangkyu.tistory.com/280)
    - 요점은 junit5이전 버전은 리플렉션 API를 사용해서 테스트 코드를 실행시켜야하기 때문에 반드시 public이여야했는데, junit5부터는 그 제약을 없앰

![movejunit4to5_2](../img/movejunit4to5_2.png)

# 검증

- mvn test를 실행
- 소나큐브 unit tests 숫자 변동 없는지 확인

![movejunit4to5_3](../img/movejunit4to5_3.png)

# Reference

- [https://subji.github.io/posts/2021/01/06/springrestdocsexample](https://subji.github.io/posts/2021/01/06/springrestdocsexample)