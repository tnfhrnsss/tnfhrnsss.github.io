---
layout: post
title: Change FeignClient to Webflux with spring load balancer
date: 2023-04-03 10:06:00
last_modified_at : 2023-04-03 10:06:00
parent: Feign
grand_parent: Msa
nav_exclude: true
---

# Note.

- 이전 post([change feignClient to reactiveFeignClient](https://tnfhrnsss.github.io/docs/msa/feign/change_feignClient_to_reactiveFeignClient/))에서 openfeign의 feign-reactor를 적용해봤는데요.
- 취약점이 있어서 제품에는 적용하지 않았습니다.([feign-reactor-spring-cloud-starter](https://mvnrepository.com/artifact/com.playtika.reactivefeign/feign-reactor-spring-cloud-starter/3.2.6))
- 대신 webflux를 직접 구현하는 것으로 방향을 바꿨습니다.
    - 제품은 spring5.x에 spring boot 2.7.x를 적용하고 있기 때문에 http interface를 사용하지 못하는데요. spring 6을 사용하고 있다면 http interface 적용도 도전해볼만합니다.
- spring cloud square가 정식으로 나온다면, 이것으로 대체가 가능할지도 모르겠습니다!!

# 환경

- java 8.x
- spring 5.x, spring boot 2.7.x

# Steps

## 1. Add dependency

```yaml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

## 2. Create bean `WebClient.Builder`

```java
@LoadBalancerClient(name="crema-gateway", configuration = {FeignLBInstanceConfigurationWithHealthCheck.class})
public class FeignLBClientConfiguration {

    @LoadBalanced
    @Bean
    WebClient.Builder webClientBuilder() {
        return WebClient.builder();
    }
}
```

- neflix-ribbon에서 scl로 업그레이드하면서 @LoadBalancerClient를 config로 올린 작업을 진행했었습니다.([spring_upgrade_scl](https://tnfhrnsss.github.io/docs/msa/spring/spring_upgrade_scl/))
- 거기에 이어서 WebClient만 bean으로 추가하면 됩니다.

## 3. Write service

```java
@Service
@RequiredArgsConstructor
public class KakaoBotHistoryFlowService {
    private final WebClient.Builder webClientBuilder;

    public void requestBotHistory(ActiveKakao activeKakao) {
        Flux<KakaoBotHistoryRdo> historyRdoFlux = webClientBuilder
            .build()
            .post()
            .uri("http://crema-gateway/chatbothistory/")
            .body(Mono.fromSupplier(() -> new KakaoBotHistorySdo(activeKakao.getUserKey(), activeKakao.getSenderKey())), KakaoBotHistorySdo.class)
            .retrieve()
            .bodyToFlux(KakaoBotHistoryRdo.class);

        historyRdoFlux.subscribe(historyRdo -> sendMessage(activeKakao, historyRdo));
    }
}
```

- bean으로 선언된 webclientbuilder를 통해 외부 챗봇이력을 non-blocking방식으로 조회한 후 메시지 어플리케이션으로 전달하는 로직입니다.
- config속성이 feignclient처럼 단순하게 분리해서 동작이 되도록 라이브러리가 탄생했음 좋겠네요

# webflux 6 이라면

- spring developer 유튜브 채널에서 interface clients in spring([https://youtu.be/lsrJu1Ul_fE](https://youtu.be/lsrJu1Ul_fE))영상을 보면 현재 feign client의 단점과 앞으로 어떤 작업을 예정하고 있다는 것과 incubator 프로젝트인 spring cloud square에 대해서 설명하고 있습니다.
- 내용 중에 `HttpServiceProxyFactory`와 `WebClientAdapter`구현체를 통해서 webclient를 쉽게 작성할 수 있는 방법이 소개되고 있어서 기록차 기재합니다.

```java
@Bean
public HttpServiceProxyFactory httpServiceProxyFactory(
		WebClient.Builder webClientBuilder, LoadBalancedExchangeFilterFunction lbFunction) {
	WebClient webClient = webClientBuilder.baseUrl("http://verification-service")
			.filter(lbFunction)
			.build();
	return HttpServiceProxyFactory.builder(WebClientAdapter.forClient(webClient))
			.build();
}
```

- [slideshare 주소](https://www.slideshare.net/secret/1rNgnJ4Ijak80x)
- [소스 코드 github](https://github.com/OlgaMaciaszek/spring-one-essentials-http-interface-clients/tree/non-reactive)

# Reference

[baeldung spring-cloud-load-balancer](https://www.baeldung.com/spring-cloud-load-balancer)

[https://happycloud-lee.tistory.com/221](https://happycloud-lee.tistory.com/221)
