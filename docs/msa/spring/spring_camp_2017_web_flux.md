---
layout: post
title: spring camp 2017 about spring web flux
date: 2022-10-03 12:50:00
last_modified_at : 2022-10-03 12:50:00
parent: Spring
grand_parent: Msa
nav_exclude: true
tags: [spring, webflux, async]
---

# 강의

[https://www.youtube.com/watch?v=2E_1yb8iLKk](https://www.youtube.com/watch?v=2E_1yb8iLKk) (발표자: 이일민)

## 스프링 5.0 webflux소개와 개발방법

- 리액티브 프로그래밍이란 무엇인가
- 클라우드에서 리액티브 시스템 구축
- reactor, rxjava 사용법

### 용도

- 비동기-논블로킹 리액티브 개발에 사용
- 고성능 웹 애플리케이션 개발
- 서비스간 호출이 많은 마이크로 서비스 아키텍처에 적합

### 비동기-논블록킹 리액티브 웹 어플리케이션의 효과를 얻으려면

- WebFlux + 리액티브 리포지토리 + 리액티브 원격API호출 + 리액티브 지원 외부 서비스 + @Async 블로킹 IO
- 코드에서 중간 코드에 블록킹 작업이 발생하지 않도록 Flux 스트림 또는 Mono에 데이터를 넣어서 전달

### 리액티브 함수형은 꼭 성능 때문만?

- 함수형 스타일코드는 편한 코드 작성
- 데이터 흐름에 다양한 오퍼레이터 적용
- 연산을 조합해서 만든 동시성 정보가 노출되지 않는 추상화 코드 작성
    - 동기, 비동기, 블록킹, 논블록킹 등을 유연하게 적용
- 데이터의 흐름의 속도를 제어할 수 있는 메커니즘 제공

### 장점 및 단점

- 장점
    - 함수형 스타일의 장점
        - 모든 웹 요청 처리 작업을 명시적인 코드로 작성
            - 메소드 시그니처 관례와 타입 체크가 불가능한 어노테이션에 의존하는 MVC스타일보다 명확
            - 정확한 타입 체크가능
        - 함수 조합을 통한 편리한 구성, 추상화 유리
        - 테스트 코드 작성의 편리함
            - 핸들러 로직 및 요청 미팽과 리턴값 처리까지 단위테스트로 작성 가능
    - 논블록킹 IO에만 효과적인것이 아니라 시스템 외부에서 발생하는 이벤트 및 클라이언트로부터의 이벤트에도 유용하다.
- 단점
    - 함수형 스타일의 코드 작성이 편하지 않으면 코드 작성과 이해 모두 어려움

### webflux를 개발하는 방식 2가지

- 기존의 @MVC 방식 - @Controller, @RestController, @RequestMapping
- 새로운 함수형 모델 - annotation을 사용하지 않고, routerfunction, handlerFunction사용

### 새로운 요청-응답 모델

- 서블릿 스택과 API에서 탈피
    - 서블릿API는 리액티브 함수형 스타일에 적합하지 않음
    - HttpServletRequest, HttpServletResponse → ServerRequest, ServerResponse인 추사황 모델로 대체

### 지원 웹 서버/컨테이너

- Servlet 3.1+ (Tomcat, Jetty, …) - 서블릿 3.1+의 비동기-논블로킹 요청 처리가능
- netty
- undertow

## 함수형 스타일 webflux(RouterFunction + HandlerFunction)이란

### 함수형 webflux가 웹 요청을 처리하는 방식 예제

1. 요청 매핑 → RouterFunction이 담당

```java
// HandlerFunction 구현
HandlerFunction helloHandler = req -> {
	String name = req.pathVariable("name");
	Mono<String> result = Mono.just("Hello " + name);
	Mono<ServerResponse> res = ServerResponse.ok().body(result, String.class); // ServerResponse의 빌더를 활용
	return res;  // Mono에 담긴 ServerResponse타입으로 리턴
}

// 또는
HandlerFunction helloHandler = req ->
	ok().body(fromObject("Hello " + req.pathVariable("name")));

----
// RouterFunction 구현
RouterFunction router = req ->
	RequestPredicates.path("/hell/{name}").test(req) ? // url경로 패턴 검사
			Mono.just(helloHandler) : Mono.empty();
```

### (1) RouterFunction

1. 함수형 스타일의 요청 매핑
    
    ```java
    @FuctionalInterface
    public interface RouterFunction<T extends ServerResponse> {
    	Mono<HandlerFunction<T>> route(ServerRequest request);
    }
    ```
    
    - webflux버전의 웹 응답인 ServerResponse이나 그 서브타입의 Mono퍼블리셔를 리턴하는 HandlerFunction의 Mono타입

### (2) HandlerFunction

- 함수형 스타일의 웹 핸들러(컨트롤러 메소드)
- 웹 요청을 받아 웹 응답을 돌려주는 함수

```java
@FuctionalInterface
public interface HandlerFunction<T extends ServerResponse> {
	Mono<T> handle(ServerRequest request);
}
```

### (3) HandlerFunction과 RouterFunction 조합을 통해 코드를 좀더 간결하게 가능하다

- RouterFunctions.route(predicate, handler)
    
    ```java
    RouterFunction router = 
    	RouterFunctions.route(RequestPredicates.path("/hello/{name}"), // test()처럼 실행시키지 않고 검사만
    		req -> ServerResponse.ok().body(fromObject("Hello " + 
    				req,pathVariable("name")))));
    ```
    

### RouterFunction의 등록

- RouterFunction타입의 @Bean으로 만든다. 람다식 오브젝트를 하나 만들어서
- 핸들러 내부 로직이 복잡하다면, 분리한다
    - 핸들러 코드만 람다식으로 따로 선언하거나
    - 메소드를 정의하고 메소드 참조로 가져온다.

```java
HandlerFunction handler = req -> {
	String res = myService.hello(req.pathVariable("name"));
	return ok().body(fromObject(res));
};

@Bean
RouterFunction helloPathVarRouter() {
	//return route(RequestPredicates.path("/hello/{name}"),
			//req -> ok().body(fromObject("Hello " + req.pathVariable("name"))));
	return route(path("/hello/{name}"), handler);
}

// 또는
@Component
public class HelloHandler {
		@Autowired MyService myService;

		Mono<ServerResponse> hello(ServerRequest req) {
			String res = myService.hello(req.pathVariable("name"));
	    return ok().body(fromObject(res));
		}
		
}

@Bean
RouterFunction helloRouter(@Autowired HelloHandler helloHandler) {
		return route(path("/hello/{name}"), helloHandler::hello); // 빈의 핸들러 메서드 레퍼런스
}
```

- 좀더 나은 방향
    1. RouterFunction의 조합으로 개선
        - 핸들러마다 Bean을 생성할 순 없으니까
            - → RouterFunction의 and(), andRoute() 등으로 하나의 Bean에 n개의 RouterFunction을 체인으로 선언한다.
    2. RouterFunction의 매핑 조건의 중복
        1. 타입레벨 - 메소드 레벨의 @RequestMapping처럼 공통의 조건을 정의하는 것 가능
            1. → RouterFunction.nest()을 사용
            
            ```java
            public RouterFunction<?> routingFunction() {
            	return nest(pathPrefix("/person"),  // /person로 시작하는 api들을 연결
            		nest(accept(APPLICATION_JSON),
            		route(GET("/{id}"), handler::getPerson)
            		.andRoute(method(HttpMethod.GET), handler::listPeople)
            	).andRoute(POST("/").and(contentType(APPLICATION_JSON)),
            	handler::createPerson));
            }
            ```
            

## @MVC WebFlux (@Controller + @RequestMapping) 개발방식

- 비동기 + 논블로킹 리액티브 스타일로 코드 작성 가능
- 매핑만 어노테이션 방식을 이용

```java
@RestController
public static class MyController {
	@RequestMapping("/hello/{name}")
	Mono<ServerResponse> hello(ServerRequest req) {
		return ok().body(fromObject(req.pathVariable("name");
	}

}
```

- @MVC 요청 바인딩과 Mono/Flux 리턴 값
    - 가장 대표적인 @MVC WebFlux 작성 방식
    - 파라미터 바인딩은 @MVC 방식 그대로
    - 핸들러 로직 코드의 결과를 Mono /Flux 타입으로 리턴
    
    ```java
    @GetMapping("/hello/{name}")
    Mono<String> hello(@PathVariable String name) {
    	return Mono.just("Hello " + name);
    }
    
    ```
    
    - @RequestBody 바인딩(json, xml)
        - T
        - Mono<T>
        - Flux<T>
        
        ```java
        @RequestMapping("/hello/{name}")
        Mono<String> hello(@PathVariable Mono<User> user) { //변환된 오브젝트를 Mono에 담아서 전달
        	return user.map(u -> "Hello" + u.getName()); // Mono의 연산자를 사용해서 로직을 수행하고 Mono로 리턴
        }
        
        // 또는 스트림으로 표현할 수 있다
        @PostMapping("/hello")
        Flux<String> hello(@PathVariable Flux<User> user) { //변환된 오브젝트를 Mono에 담아서 전달
        	return user.map(u -> "Hello" + u.getName()); // User의 스트림 형태로 로직 수행
        }
        
        ```
        
    - @ResponseBody 리턴 값 타입
        - T
        - Mono<T>
        - Flux<T>
        - Flux<ServerSideEvent> : Httpstream으로 리턴하고 싶을 때
        - void
        - Mono<Void>

## WebFlux와 리액티브 기술(WebClient + Reactive Data)

Q. WebFlux만으로 성능이 좋아질까?

- 비동기-논블로킹 구조의 장점은 블록킹 IO를 제거하는데서 나온다
- HTTP서버에서 논블로킹 IO는 오래 전부터 지원
- 이것만으로는 성능이 나아지지 않는다. 다른 것도 개선해야하는데

### 개선할 블로킹 IO

- 데이터 액세스 리포지토리
    - JPA - JDBC기반 RDB연결
        - 현재는 답이 없다
        - 일부 DB는 논블로킹 드라이버 존재
        - @Async 비동기 호출과 CFuture를 리액티브로 연결하고 쓰레드풀 관리를 통해서 웹 연결 자원을 효율적으로 사용하도록 만드는 정도
            - 물론 db접속은 블록킹이지만 아래처럼하면 db로 가져오는 것 외에는 논블록킹으로 구현할 수 있다.
            
            ```java
            @Async
            CompletableFuture<User> findOneByFirstname(String firstname);
            
            // 이렇게 만들고 이걸 여기서 Mono타입으로 호출
            @GetMapping
            Mono<User> findUser(String name) { // fromCompletionStage로 리턴시키면 비동기적
            	return Mono.fromCompletionStage(myRepository.findOneByFirstName(name));
            }
            ```
            
        - Async JDBC가 등장한다면 해결
- HTTP API 호출
- 기타 네트워크를 이용하는 서비스
- 본격 리액티브 데이터 액세스 기술
    - 스프링 데이터의 리액티브 리포지토리 이용
        - MongoDB
        - Cassandra
        - Redis
    - ReactiveCrudRepository 확장
        
        ```java
        public interface ReactivePersonReposityro extends ReactiveCrudRepository<Person, String> {
        	Flux<Person> findByLastname(Mono<String> lastname);
        
        	@QUery("{'firstname':?0, 'lastname;:?1}")
        	Mono<Person> findByFirstnameAnd Lastname(String firstname, String lastname);
        	// 0 또는 1개의 결과를 비동기-논블로킹 리액티브로 조회하도록 결과를 Mono<T>로 리턴
        }
        	
        ```
        
- 논블로킹 API호출은 WebClient
    - AsyncRestTemplate의 리액티브 버전
    - 요청을 Mono/Flux로 전달할 수 있고
    - 응답을 Mono/Flux형태로 가져온다.
    
    ```java
    @GetMapping("/webclient")
    Mono<String> webclient() {
    	return WebClient.create("http://localhost:8080")
    			.get()
    			.uri("/hello/{name}", "Spring")
    			.accept(MediaType.TEXT_PLAIN)
    			.exchange()
    			.flatMap(r -> r.bodyToMono(String.class)) // 요청 바디를 String타입으로 변환해서 Mono에 
    //담아 리턴하는 함수로 Mono데이터에 적용한 함수의 결과가 Mono타입이기 때문에 flatMap을 적용해야한다. 아니면
    // Mono<Mono<String>>이 됨
    			.map(d -> d.toUpeerCase())
    			.flatMap(d -> helloRepository.save(d));
    }
    ```
    

## 공부해야할 것

- 함수형 프로그래밍
- CompletableFuture와 같이 비동기 작업의 조합, 결합에 뛰어난 툴의 사용법 익힐 것
- ReactCore 학습
    - Mono/Flux, 오퍼레이터, 스케쥴러
- WebFlux와 스프링의 리액티브 스택 공부
- 비동기 논블로킹 성능과 관련된 벤치마킹, 모니터링, 디버깅 연구
- 테스트