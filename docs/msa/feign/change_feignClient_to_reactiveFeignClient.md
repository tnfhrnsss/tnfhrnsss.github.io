---
layout: post
title: Change FeignClient to ReactiveFeignClient
date: 2023-02-04 22:03:00
last_modified_at : 2023-02-04 22:03:00
parent: Feign
grand_parent: Msa
nav_exclude: true
---

# note

- 기존에는 feign을 통해 restapi를 호출할 경우,
    - 업스트림에 대기가 길어질 때를 대비해 timeout을 설정해서 fast-fail을 유도하도록 했었습니다.
    - spring 풀이 아닌 별도의 was pool을 생성해서 async나 completedfuture를 적절히 혼합해서 호출하도록 했습니다.
- 모든 기능을 비동기 non blocking으로 하는 것보다 요청량이 많고 비지니스 부하가 예상되는 곳, 비지니스 순서가 중요하지 않은 곳에만 적용하는 것이 좋습니다.
- 단점으로는 추적이 쉽지 않다는 것, fast-fail을 기대할 수 없다는 것..이 있겠습니다

# summary

- 현재 openfeign나 spring-cloud-openfeign에는 reactive client를 지원하고 있지 않습니다.
    - openfeign의 경우 reactive stream wrapper를 통해서 약간은? reactive하게 만들 수 있습니다. 하지만 이것도 결국은 blocking이라는 것..[https://github.com/OpenFeign/feign/tree/master/reactive](https://github.com/OpenFeign/feign/tree/master/reactive){:target="_blank"}
- 그래서 spring webclient를 통해 feignclient를 구현하려면 feign-reactive를 사용하는 방법과 직접 webclient를 통해 호출하는 방법이 있습니다. 여기서는 feign-reactive를 사용하려고 합니다.
- playtikaOSS의 feign-reactive를 통해 구현합니다.

# step.

1. dependency추가

```xml
<dependency>
    <groupId>com.playtika.reactivefeign</groupId>
    <artifactId>feign-reactor-spring-cloud-starter</artifactId>
    <version>3.2.6</version>
</dependency>
```

1. @SpringBootApplication이 선언된 클래스에 @EnableReactiveFeignClients 선언
    1. auto scan을 통해 @ReactiveFeignClient를 찾아서 bean으로 등록하겠지만, basePackages를 통해 패키지 경로를 적어줍니다.
    
    ```java
    @EnableReactiveFeignClients(basePackages = "spectra.attic")
    @SpringBootApplication(scanBasePackages = "spectra.attic")
    public class CremaApplication {
    ```
    
2. @FeignClient를 @ReactiveFeignClient로 변경
    - contextId 제거
    
    ```java
    @ReactiveFeignClient(
        url = "${crema.thirdparty.kakao.url.chat}",
        name = "${crema.gateway.name}",
        configuration = {FeignConfiguration.class, FeignRetryConfiguration.class},
        primary = false
    )
    public interface KakaoBotHistoryClient {
    ```
    

1. Reactor 객체로 반환값 변경
    1. mono객체
    
    ```java
    @PostMapping("/messages")
    Mono<KakaoBotHistoryRdo> find(@RequestBody KakaoBotHistorySdo botHistorySdo);
    ```
    

1. 서비스단 변경
    1. 반환값을 가져옵니다. 주의할 점은 block()을 쓰면 reactor를 적용한 의미가 없습니다.
    
    ```java
    public void send(ActiveKakao activeKakao) {
        Mono<KakaoBotHistoryRdo> rdo = botHistoryClient.find(new KakaoBotHistorySdo(activeKakao.getUserKey(), activeKakao.getSenderKey()));
        rdo
    			.doOnNext(message -> {
            List<KakaoBotMessage> message = new ArrayList<>();
            for (OpenbuilderMessage openbuilderMessage : message.getChatbot_messages()) {
                message.addAll(OpenbuilderMessage.toMessageDomain(openbuilderMessage));
            }
            sendMessage(activeKakao, new KakaoBotMessageRdoListRdo(message));
            
        } )
    			.subscribe();
    }
    ```
    

## WebClient를 사용할 경우

- [https://www.baeldung.com/spring-boot-feignclient-vs-webclient](https://www.baeldung.com/spring-boot-feignclient-vs-webclient){:target="_blank"}를 참고합니다.
    
    ```java
    @GetMapping(value = "/products-non-blocking", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<Product> getProductsNonBlocking() {
        log.info("Starting NON-BLOCKING Controller!");
    
        Flux<Product> productFlux = WebClient.create()
          .get()
          .uri(getSlowServiceBaseUri() + SLOW_SERVICE_PRODUCTS_ENDPOINT_NAME)
          .retrieve()
          .bodyToFlux(Product.class);
    
        productFlux.subscribe(product -> log.info(product.toString()));
    
        log.info("Exiting NON-BLOCKING Controller!");
        return productFlux;
    }
    ```
    

## 기존 blocking방식의 restapi와의 동작방식 차이

- [java-springboot-blocking-vs-non-blocking-rest-api](https://vimalma1093.medium.com/java-springboot-blocking-vs-non-blocking-rest-api-implementation-fe5643840287){:target="_blank"}에서 매우 자세히 설명하고 있습니다.
- 스프링 캠프 유튜브에서도 자세히 들을 수 있습니다.

# reference

- [https://cloud.spring.io/spring-cloud-openfeign/reference/html/#reactive-support](https://cloud.spring.io/spring-cloud-openfeign/reference/html/#reactive-support){:target="_blank"}
- [https://www.vinsguru.com/spring-webclient-with-feign/](https://www.vinsguru.com/spring-webclient-with-feign/){:target="_blank"}
- [http://gunsdevlog.blogspot.com/2020/09/reactive-streams-reactor-webflux.html](http://gunsdevlog.blogspot.com/2020/09/reactive-streams-reactor-webflux.html){:target="_blank"}
- [https://techblog.woowahan.com/2619/](https://techblog.woowahan.com/2619/){:target="_blank"}