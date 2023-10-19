---
layout: post
title: optimization of test case
date: 2023-10-12 22:27:00
last_modified_at : 2023-10-12 22:27:00
parent: TestCase
grand_parent: Quality Practice
nav_exclude: true
---

# 테스트 코드 최적화

# 관련 강의

[https://youtu.be/N06UeRrWFdY?si=hLPQf7XNl9_vUFcG](https://youtu.be/N06UeRrWFdY?si=hLPQf7XNl9_vUFcG)

[https://youtu.be/QsD1hCzaGCU?si=lE-sqy_Kcw4Wmnyg](https://youtu.be/QsD1hCzaGCU?si=lE-sqy_Kcw4Wmnyg)

# 배경

- test first 원칙
- unit 테스트 케이스 수가 적다.
- unit 테스트뿐이다.
- 테스트 코드 수행 속도 개선이 필요하다.

# 주의
- TDD를 실패하는 사람이 하는 테스트 (출처 : okkycon:2018 이규원 - 당신들의 TDD가 실패하는 이유)
    * 코드가 이루고자 하는 가치나 기능을 테스트하기보다 그 기능을 어떻게 구현하고 있는지를 테스트한다.
    * 결국 테스트 케이스들이 구현체와 결합도가 높아진다.
    * 구현체들을 리팩토링하면 결합되어있는 테스트 케이스들이 모두 꺠져버린다.

# 작업

## 1. Unit 테스트 케이스 추가

- 46건 → 295건으로 추가 작성 진행

## 2. Junit 테스트 뿐이다.

### 2-1. Acceptance Test (계획 중)

- **User Acceptance Test (UAT - 사용자 수용 테스트)**
- **Business Acceptance Test (BAT - 비즈니스 수용 테스트)**

### 2-2. WebMvc Test

- MockMvc객체를 통해서 컨트롤러단의 테스트 케이스를 작성한다.
- 예)
    
    ```java
    
    ```
    

### 2-3. Service Integration Test(진행중)

- 통합 테스트는 다른 시스템 구성 요소 또는 모듈 간의 상호 작용을 테스트하는 것으로 이러한 테스트는 시스템의 여러 부분이 함께 작동하는지 확인하는 것이 중요하다
- 대상은 jpa, cache, kafka 등등
- 대상 1) cacheManager Test

```java
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest(
    classes = MessengerConnectorApplication.class,
    webEnvironment = SpringBootTest.WebEnvironment.MOCK
)
@ActiveProfiles({"test", "local"})
public class KakaoEndWaitingIntegrationTest {
    @Autowired
    CacheManager cacheManager;

    @Autowired
    KakaoEndWaitingServiceImpl kakaoEndWaitingService;

    @Before
    public void setUp() {
        kakaoEndWaitingService.enrol(KakaoEndWaitingSdo.sample());
        kakaoEndWaitingService.remove(KakaoEndWaitingSdo.sample().getConversationId());
    }

    @Test
    public void find() {
        String conversationId = kakaoEndWaitingService.find(KakaoEndWaitingSdo.sample().getConversationId());
        assertEquals(conversationId, getCachedBook(KakaoEndWaitingSdo.sample().getConversationId()).get());
    }

    private Optional<String> getCachedBook(String title) {
        return ofNullable(cacheManager.getCache("messengerconnector-end-kakao-find")).map(c -> c.get(title, String.class));
    }
}
```

- reference
    - [https://www.baeldung.com/spring-data-testing-cacheable](https://www.baeldung.com/spring-data-testing-cacheable)

### 2-4. DataJpa Test (계획 중)

### 2-5. kafka event Test (진행 중)

### 2-6. HttpClient Test (진행 중)

# 3. 테스트 코드 수행 속도 개선

1. @DirtiesContext 제거하기
- Application context를 계속 생성해서 로드하는 비용 제거하기 위해 @DirtiesContext 를 선언해서 재사용성을 노린 것인데 오히려 느려지게 되어 이것은 꼭 필요한 경우에만 사용하도록 한다.

1. Application Context 캐싱 조건 파악하기
    1. test 돌릴때마다 spring context를 매번 생성하는 부분 제거
        1. Spring Boot 로그가 반복적으로 로깅되고 있었다.
        2. 원인
            1. 스프링은 테스트시 application context 설정에 따라 context를 캐싱한다. 따라서 application context 설정이 다른 테스트를 요청하면 기존의 context를 활용하지 않고 새로운 context를 생성해서 사용한다.
    2. @MockBean을 생성하면 스프링은 서로 다른 컨텍스트라고 보고 application context를 매번 생성한다. 그 이유는 Mocking으로 인해서 빈 설정이 달라질 수 있기 때문이다. 이러한 경우 각 configuration파일로 분리하고 mockbean을 제거한다.
        
        ```java
        @Import(InfrastructureTestConfiguration.class)
        @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
        @ActiveProfiles("test")
        public abstract class AcceptanceTest {
        }
        
        // 
        public class PostAcceptanceTest_likeUsers extends AcceptanceTest {
        
        	
        }
        ```
        
    - WebMvcTest에서도 그와같은 application context 재사용이 안되고 계속 생성되고 있었다.
        - 해결방법 : 모든 webmvctest에서 사용하는 mock클래스와 설정 annotation들을 모아놓은 추상 클래스를 생성하고 이를 모두 상속받도록 한다.
        - 이는 webmvctest가 controller를 테스트함에 있어서 서로 다른 서비스에 대해서 간섭하지 않기 때문에 가능한 것
        
        ```java
        @RunWith(SpringRunner.class)
        @WebMvcTest(controllers = {
            // conversation
            ConversationResource.class,
            ConversationSettingResource.class,
        })
        @Import({
            ConversationConfig.class,
        })
        @AutoConfigureRestDocs
        @TestPropertySource(properties = {
            "eureka.client.enabled=false",
            "spring.cloud.config.discovery.enabled=false",
            "spring.cloud.config.enabled=false"
        })
        public abstract class MessengerConnectorResourceConfigTest extends AbstractResourceTest {
        }
        ```
        
        - MockBean도 하나의 Config파일에서만 선언하도록 했다.
            
            ```java
            @Configuration
            public class ConversationConfig {
                @MockBean
                private ConversationFlowService conversationFlowService;
            
                @MockBean
                private ConversationSettingFlowService conversationSettingFlowService;
            }
            ```
            
        - 적용 후
            - spring boot 로깅이 23번 반복되던 부분이 3번으로 줄었다.
            - 실행속도는 50초에서 48초 정도로 큰 차이는 없었다.
        
2. 테스트용 Fixture환경을 재사용하기
    1. 인수테스트시 실제로 서비스를 구동해서 http호출을 해야하기 때문에 시간이 오래걸리는 단점이 있다.

# Reference

https://tech.pick-git.com

https://bperhaps.tistory.com