---
layout: post
title: Feign client Async call
date: 2022-08-03 21:48:00
last_modified_at : 2023-11-26 08:54:00
parent: Feign
grand_parent: Msa
nav_exclude: true
tags: [feign, async]
---

## try1. Use `AsyncFeign`

[How to build REST API client using Feign in Spring Boot](https://levelup.gitconnected.com/how-to-build-rest-api-client-using-feign-in-spring-boot-50db38289420)

```java
AsyncFeign<Object> asyncFeign = AsyncFeign.asyncBuilder()
    .requestInterceptor(new FeignConfiguration().requestInterceptor())
    .build();

asyncFeign.newInstance(new Target.HardCodedTarget(BotHistoryClient.class, "test", ""));
```

구현을 하고 보니, AsyncFeign에 `@Experimental`가 있었다.  

제품에는 못 넣을꺼 같아서 직접 구현했다.

## try2. Call CompletableFuture.runAsync

```java
private final Executor taskExecutor;

public void register(ConversationMessage conversationMessage) {
		CompletableFuture.runAsync(() -> botHistoryClient.register(BotMessageBulkCdo.toDomain(conversationMessage)), taskExecutor)
            .exceptionally(throwable -> {
                log.error(throwable.getMessage());
                return null;
            });
}
```

일단 서비스 안에서 직접 completableFuture를 선언해서 비동기로 호출해봄

저는 executor를 bean으로 등록해둔 것이 있어서, commonPool을 사용하지 않음

## try3. 인터페이스 구현

요청이 많을 경우 병목이 예상되는 구간이 몇 군데 있어서 공통 인터페이스를 구현했다.

### feign client async interface 생성

```java
public interface FeignAsyncClient {
    void execute(Object o);
}
```

비동기로 feign 호출하기 위한 feignclient 인터페이스의 공통함수 선언

### feign client 구현체

```java
@FeignClient(
    contextId = "BotHistoryClient",
    name = "test",
    configuration = {FeignConfiguration.class, FeignRetryConfiguration.class},
    primary = false
)
public interface BotHistoryClient extends FeignAsyncClient {
    @Override
    @PostMapping("bothistory/messages/bulk")
    void execute(@RequestBody Object o);
}
```

비동기 호출을 해야하는 feign은 FeignAsyncClient를 상속받아서 FeignClient API를 선언하고

### 비동기 메소드 인터페이스

```java
public interface FeignAsyncHandler {
    CompletableFuture<Void> send(Object o);
}
```

서비스단에서 feignclient를 비동기 호출하기 위한 메소드를 인터페이스로 생성

### 비동기 메소드 인터페이스 구현체

```java
public class BotHistoryClientAsyncHandler implements FeignAsyncHandler {
    private final FeignAsyncClientService feignAsyncClientService;

    @Override
    public CompletableFuture<Void> send(Object o) {
        return feignAsyncClientService.apiFuture(botHistoryClient, o);
    }
}
```

send메소드를 상세구현

### FeignClient 비동기 호출

```java
public class FeignAsyncClientService {
    private final Executor taskExecutor;

    public CompletableFuture<Void> apiFuture(FeignAsyncClient feignAsyncClient, Object o) {
        return CompletableFuture.runAsync(() -> feignAsyncClient.execute(o), taskExecutor)
            .exceptionally(throwable -> {
                log.error(throwable.getMessage());
                return null;
            });
    }

    public <T> CompletableFuture<T> apiFutureSupplier(FeignAsyncSupplierClient feignAsyncClient, Object o, Class<T> rt) {
        return CompletableFuture.supplyAsync(() -> feignAsyncClient.execute(o), taskExecutor)
            .thenApply(response -> JsonUtil.fromJson(response, rt));
    }

}
```

* FeignAsyncClientService에서 runAsync 또는 리턴용 supplyAsync로 비동기 호출하도록 한다.

*IMPOTANT* 
* (23.11.25) 리턴용 함수 apiFutureSupplier를 수정
    - 기존에는 get()함수를 통해 응답값을 반환받다보니, 결과를 기다리게 되고, 결국  비동기적으로 호출했음에도 불구하고 블록킹이 발생해서 병목이 발생되는 구간이 되고 말았다. 
    - CompletableFuture<T> 로 반환하고, 호출한 서비스에서 아래와 같이 결과 또는 예외처리를 한다.
    - 고려할 점은 get을 통해 병목이 발생해서, 비동기 처리로 병목이 해결된다해도 다른 곳으로 부하가 이전할 가능성이 있기 때문에 부하 해결을 위해서 비동기처리를 한다고 생각하는 것이 항상 맞는 것은 아니다.

    ```java
        public void write(String conversationId, KakaoWriteCdo kakaoWriteCdo) {
            this.send(kakaoWriteCdo).thenAccept(kakao -> whenComplete(kakao, conversationId));
        }

        public CompletableFuture<KakaoClientRdo> send(Object o) {
            try {
                return feignAsyncClientService.apiFutureSupplier(kakaoWriteClient, o, KakaoClientRdo.class);    
            } catch (RuntimeException e) {        
                if (ExceptionUtils.containsExceptionNested(e, ConnectException.class)) {
                    log.error("Kakao Connection failed: Connection refused.");        
                }    
            }    
            return CompletableFuture.completedFuture(KakaoClientRdo.defaultError());
        }

    ```


