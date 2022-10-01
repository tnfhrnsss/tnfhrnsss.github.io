---
layout: post
title: change Hystrix to Resilience4j
date: 2022-10-01 09:48:00
last_modified_at : 2022-10-01 09:48:00
parent: SpringCloud
grand_parent: Msa
nav_exclude: true
tags: [spring cloud, spring boot]
---

# change note

- exception에 대한 처리 변경

## 1. change dependency

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

## 2. change code

### HystrixRuntimeException → `RuntimeException`

- 기존에는 FeignClient로 호출하고 발생하는 Exception에 대한 폴백처리를 HystrixRuntimeException를 통해서 하고 있었는데, 이 부분이 사라지면서 java의 `RuntimeException`으로 대체했습니다

### feign.hystrix.FallbackFactory → org.springframework.cloud.openfeign.FallbackFactory

- 기존에는 feign.hystrix.FallbackFactory를 사용해서 Feign호출 후 Fallback처리를 하도록 했는데, org.springframework.cloud.openfeign.FallbackFactory로 변경

### status code

- 다른 어플리케이션으로 feign call했는데, 가용한 상태가 아니라면,
    - feign.RetryableException를 throw하고
    - status를 -1 로 리턴시킨다

```java
throwable = {RetryableException@21177} "feign.RetryableException: "
 retryAfter = null
 httpMethod = {Request$HttpMethod@21226} "PUT"
 status = -1
 responseBody = null
 responseHeaders = null
 request = {Request@21227} 
 detailMessage = ""
 cause = {UnknownHostException@21229} "java.net.UnknownHostException: espresso"
 stackTrace = {StackTraceElement[9]@21233}
```

### 최종 코드

- FeignClient
    - fallbackFactory를 선언하고, BusinessFallbackFactory를 상속받는다
    
    ```java
    @FeignClient(
        contextId = "ReceptionClient",
        name = "test",
        configuration = {FeignConfiguration.class, FeignLoggerLevelConfiguration.class},
        fallbackFactory = ReceptionClient.ReceptionFallback.class,
        primary = false
    )
    public interface ReceptionClient {
        @PostMapping("receptions")
        ReceptionIdRdo register(@RequestBody ReceptionCdo receptionCdo);
    
        @Component
        class ReceptionFallback extends BusinessFallbackFactory {
            public ReceptionFallback(BusinessExceptionConverter businessExceptionConverter) {
                super(businessExceptionConverter);
            }
    
            @Override
            public ReceptionClient create(Throwable throwable) {
                return (partitionId, receptionCdo) -> {
                    warning(new BusinessFallbackResponse(receptionCdo.getReceptionId(), throwable));
                    return ReceptionIdRdo.EMPTY;
                };
            }
        }
    }
    ```
    
- FallbackFactory Class
    - FallbackFactory을 상세 구현한다.
    - 여러 client에서 공통으로 사용하게 하기 위해 상세 구현체는 별도로 분리했다.
    
    ```java
    @Slf4j
    @Component
    @RequiredArgsConstructor
    public abstract class BusinessFallbackFactory implements FallbackFactory<Object> {
        private final BusinessExceptionConverter businessExceptionConverter;
    
        public void warning(BusinessFallbackResponse businessFallbackResponse) {
            businessExceptionConverter.accept(businessFallbackResponse);
        }
    
        public void warning(Throwable throwable) {
            log.error("A feign failed error occurred. " + throwable.getMessage());
        }
    }
    ```
    
- ExceptionConverter
    - FeignException status별로 실패 비지니스가 다르기 때문에, case처리했다.
    
    ```java
    @Slf4j
    @Component
    @RequiredArgsConstructor
    public class BusinessExceptionConverter implements Consumer<Object> {
        private final BusinessMessageAdapter businessMessageAdapter;
    
        @Override
        public void accept(Object obj) {
            action((BusinessFallbackResponse) obj);
        }
    
        @SuppressWarnings("squid:S1301")
        private void action(BusinessFallbackResponse response) {
            Throwable e = response.getThrowable();
            if (e instanceof FeignException) {
                FeignException exception = (FeignException) e;
                switch (exception.status()) {
                    case 400001 :
    						}
    				}
    		}
    }
    ```