---
layout: post
title: Async & Spring 개발
date: 2022-10-24 00:31:00
last_modified_at : 2022-10-24 00:31:00
parent: Spring
grand_parent: Msa
nav_exclude: true
tags: [spring, async]
---

## 강의 주소

[https://www.youtube.com/watch?v=HKlUvCv9hvA](https://www.youtube.com/watch?v=HKlUvCv9hvA)

## 비동기 개발을 위한 필요한 사전지식

1. 자바 비동기 개발
    1. 비동기와 논블록킹
    2. java5+ : Future/Executor(s), BlockingQueue
    3. java7 : ForkJoinTask
    4. java8 : CompletableFuture
    5. java9 : Flow
2. 서블릿 비동기 개발
    1. 스프링이 서블릿 스택 기반으로 동작하기 때문에 알아야 함
    2. Servlet 3.0 Async Processing : AsyncContext
    3. Servlet 3.1 : Non-Blocking IO
    4. Servlet 4.0
3. 스프링 비동기 개발
    1. @Async
    2. Async Request Processing
        1. Callable
        2. DeferedResult
        3. ResponseBodyEmitter
    3. AsyncRestTemplate

## 스프링 3.2 ~ 4.3에서의 비동기 개발 방법

## @Async

- Async로 호출한 메서드를 String으로 리턴받고자 하면 null로 리턴한다. @Async가 지원하는 리턴타입은
    - void
    - Future<T>
        - 리턴받을 타입을 감싸서 호출
        - 비동기 결과를 가져올 수 있는 기본 채널 인터페이스를 정의한 것
            
            ```java
            @Async
            Future<String> service() {
            	return new AsyncResult<>(result);
            }
            
            Future<String> f = myService.service();
            String res = f.get();
            ```
            
    - ListenableFuture<T>
        - spring에서 표준화시킨것
        - Future에서 .get할때 블록킹하는 것을 ListenableFuture를 통해 논블로킹으로 가져올 수 있다는 장점이 있다.
        
        ```java
        @Async
        ListenableFuture<String> service() {
        	return new AsyncResult<>(result);
        }
        
        ListenableFuture<String> f = myService.service();
        f.addCallback(r -> log.info("Success: {}", r), e -> log.info("Error: {}", e));
        
        ```
        
    - CompletableFuture<T>
        - java8이후에 등장
        
        ```java
        @Async
        CompletableFuture<String> service() {
        	return CompletableFuture.completedFuture(result);
        }
        
        CompletableFuture<String> f = myService.service();
        f.thenAccept(r -> System.out.println(r));
        ```
        
- SimpleAsyncTaskExecutor
    - @Async를 남발하는 경우가 많은데, @Async가 사용하는 기본 TaskExecutor
    - @Async를 아무런 설정없이 그냥 사용하게 되면 그런 경우는 비동기 쓰레드는 SimpleAsyncTaskExecutor라는 스프링에서 자동으로 할당한 bean에서 가져오게 된다. SimpleAsyncTaskExecutor의 단점은 쓰레드 재사용이 아닌 새로운 쓰레드를 호출할 때 마다 만든다. 이렇게 사용하는 쓰레드는 낭비적이게 된다.
    - @Async를 본격적으로 사용한다면, 실전에선 사용하지 말 것
        - 사용한다고 하면, 다음 타입의 빈을 하나만 등록을 하면, @Async(”myExecutor”) 처럼 호출할 때 사용하게 한다.
            - Executor
            - ExecutorService
            - TaskExecutor
            - 위의 3개 중 하나만 등록해야한다. 여러개하면 에러가 날 것

```java
@Bean
TaskExecutor taskExecutor() {
	ThreadPoolTaskExecutor te = new ThreadPoolTaskExecutor();
	te.setCorePoolSize(10);
	te.setMaxPoolSize(100);
	te.setQueueCapacity(50);
	te.initialize();
	return te;
}

// 50개의 @Async메서드 호출이 동시에 일어나면 쓰레드는 몇개가 만들어질까? - 10개
-- 쓰레드 풀은 기본적으로 corepoolsize를 초과하면 maxpoolsize까지 늘리는게 아니라 먼저 queuesize만큼 
queue를 채운다. 그래서 queue를 50개까지 채우고 기다렸다가 그리고도 초과하면 maxpoolsize만큼 늘린다.
```

## Asynchronous Request Processing

- 등장 배경
    - Servlet 3.0 - AsyncContext
        - 무엇인가 기다리느라 점유하는 쓰레드를 해결하고자 서블릿 요청처리를 완료하지 못하는 경우를 위해서 등장
        - 서블릿에서 AsyncContext를 만든 뒤 즉시 서블릿 메서드 종료 및 서블릿 쓰레드 반납
        - 어디서든 AsyncContext를 이용해서 응답을 넣으면 클라이언트에 결과를 보냄 - AsyncContext를 저장해두고 임의의 쓰레드에서 응답 결과를 넣을 수 있다.
            
            ```java
            @WebServlet(urlPattern = "/hello", asyncSupported = true)
            public class MyServlet extends HttpServlet {
            	public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
            		AsyncContext ac = request.startAsync();
            		Executors.newSingleThreadExecutor().submit(() -> {
            				ac.getResponse().getWriter().println("Hello Servlet");
            				ac.complete();
            				return null;
            		});
            	}
            }
            ```
            
    - Servlet 3.1 - Non-blocking IO
        - 요청 읽기와 응답 쓰기에 콜백을 이용
        - ReaderListener
        - WriterListener
- 비동기 스프링 @MVC
    - Servlet 3.0+ 비동기 요청 처리 기반의 @MVC
    - 비동기 @MVC의 리턴 타입
        - Callable<T>
            - AsyncTaskExecutor에서 실행될 코드를 리턴
            - new로 thread를 생성하지 않고, 쓰레드 안에서 비동기로 동작할 코드를 Callable인터페이스를 통해서 동작하도록  하는 것
            - Callable이란 파라미터는 있고 리턴값은 없는 것
            
            ```java
            @FunctionInterface
            public interface Callable<V> {
            	V call() throws Exception; 
            }
            
            // 적용하면
            @GetMapping("/callable")
            Callable<String> callable() {
            	return () -> {  // 컨트롤러 메소드가 종료된 뒤 별도의 TaskExecutor내에서 실행, 즉 spring이 갖고 있는 thread pool위에서 실행시켜준다 
            		return "hello"; // 완료가 되면, Callable의 리턴값이 컨트롤러 메서드의 리턴 값으로 사용
            	};
            }
            ```
            
        - WebAsyncTask<T>
            - Callable과 동일한데, Callable에 없는 2개의 api가 추가되었다
                - timeout
                - taskExecutor
                
                ```java
                public WebAsyncTask(Long timeout, String executorName, Callable<V> callable) {}
                public WebAsyncTask(Long timeout, AsyncTaskExecutor executor, Callable<V> callable) {}
                
                // 적용 예
                @GetMapping("/webasynctask")
                WebAsyncTask<String> webasynctask() {
                	return new WebAsyncTask<String>(5000L, "myAsyncExecutor",
                			() -> {
                				return "hello";
                			}
                }
                ```
                
        - DeferredResult<T>
            - 임의의 쓰레드에서 리턴 값 설정 가능
            - Callable처럼 새로운 쓰레드를 만들지 않음
            - 다양한 비동기 처리 기술과 손쉽게 결합
            - api response body에 대한 핸들러를 만들 수 있다.
            - 지연된 결과를 만들어 낼 수 있다. 매우 유용하다.
                
                ```java
                public class DeferredResult<T> {
                	public boolean setResult(T result) {}
                	public boolean setErrorResult(Object result) {}
                }
                
                // 적용 예
                @GetMapping("/deferredresult")
                DeferredResult<String> deferredResult() {
                	DeferredResult dr = new DeferredResult();
                	queue.add(dr);
                	return dr; // 아직 api를 호출할 쪽에는 결과가 리턴되지 않고 쓰레드에 쓴다. 다른 쓰레드에서 저장된 DeferredResult에 결과 값을 쓴다.
                }
                
                void eventHandler(String event) {
                	queue.forEach(dr -> dr.setResult(event)); // 이때 쓰레드를 쓰면서
                }
                ```
                
            - DeferredResult와 Async결합
                - @Async와 같이 쓰면 좋다. @Async메소드가 리턴하는 ListenableFuture에서 DeferredResult사용
                - 비동기 @MVC + 비동기 메소드 실행
                
                ```java
                @GetMapping("drlf")
                DeferredResult<String> drAndLf() {
                	DeferredResult dr = new DeferredResult();
                
                	ListenableFuture<String> if = myService.async();  // 비동기 작업을 시작한 뒤, 결과에 대한 핸들러만 받는다. 
                	if.addCallback(r -> dr.setResult(r), e -> dr.setErrorResult(e)); // 비동기 작업이 끝나면 실행될 콜백에서 지연된 @MVC결과값을 등록한다.
                
                	return dr;
                }
                ```
                
        - ListenableFuture<T>
            - 스프링4.x부터 등장한 것으로 DeferredResult와 Async결합방법 대신 쓸 수 있다. 그래서 DeferredResult생성, 콜백 등록과 콜백 호출시 결과를 넘기는 것까지 스프링이 알아서 해준다.
                
                ```java
                @GetMapping("/if")
                ListenableFuture<String> listenableFuture() {
                	return myService.async();
                }
                ```
                
            - 한계점
                - 두가지 이상의 비동기 작업을 순차적으로 혹은 동시에 수행하고 결과를 조합해서 @MVC의 리턴값으로 넘기는 것은 안된다. 콜백이란 결과를 만들어내야하기 때문에
            - 한계점 개선 방법
                - ListenableFuture<T> 조합
                    - 방법
                        - 두 개 이상의 비동기 작업을 결합할 때는 다시 콜백 + DeferredResult방식으로 구현한다.
                        - 비동기 작업의 성공 콜백에서 다음 비동기 작업을 중첩적으로 넣어서 시도
                        - 최종 비동기 작업의 성공 콜백에서 DeferredResult에 결과 전달
                    - 여러개의 비동기 작업을 조합해서 비동기 @MVC의 결과로 사용할 수 있다. 하지만 콜백의 중첩으로 코드가 복잡해지고, 예외 콜백의 내용이 동일할 경우 중복이 발생한다.
            - CompletableFuture<T> 조합
                - java8에서 위에서 말한 중첩 ListenableFuture<T> 조합의 방식의 단점을 개선하고자 나온 것
                - 함수형 스타일 접근방법
                - CompletionStage의 서브타입
                - CompletionStage의 체이닝으로 간결하게 표현
                    
                    ```java
                    @GetMapping("composecf")
                    CompletableFuture<String> cfCompose() {
                    	//CompletableFuture<String> f1 = myService.casync();
                    	//CompletableFuture<String> f2 = f1.thenCompose(res1 -> myService.casync2(res1));
                    	//return f2;
                    
                    	return myService.casync()
                    				.thenCompose(res1 -> myService.casync2(res1));
                    }
                    ```
                    
                - 중복되는 예외처리를 한번에
                - 다양한 비동기/동기 작업의 변환, 조합, 결합 가능
                - CompletableFuture/CompletionStage사용에 대한 학습이 필요
                - ExecutorService의 활용기업도 익혀야 함
            - 비동기 작업의 결합
                - 2개 이상의 비동기 작업을 병렬적으로 실행하고 결과를 모아서 결과 값을 만들어 내는 비동기 작업 구성으로
                    - ListenableFuture의 콜백 구조로는 어렵다
                    - CompletableFuture로는 그런 구성이 쉽게 가능하다.
        - CompletionStage<T>
        - ResponseBodyEmitter

## AsyncRestTemplate

- 비동기 논블로킹 API호출
    - 블로킹이 많이 일어나는 외부 API호출들이나 사내의 다른 서비스 호출하는 것들에서 활용
- 배경
    - 비동기 논블로킹 API 요청과 @MVC
        - RestTemplate은 동기-블로킹 방식이라 API호출작업 동안 쓰레드 점유
        - 블로킹으로 인한 컨텍스트 스위칭이 두번 발생(호출할 때 응답할때)해서 비효율적
        - 비동기 @MVC를 사용했다고 하더라도 쓰레드 자원의 효율적인 사용이 어려움
- 스프링4.0부터 사용할 수 있다.

```java
AsyncRestTemplate art = new AsyncRestTemplate();
for (int i = 0; i < 100; i++) { // 100개의 API를 호출하는데 1초 걸리지만, 쓰레드를 100개를 만들어버려서 
	ListenableFuture<ResponseEntity<String>> if = art.getForEntity("http://localhost:8080/api, String.class);
	if.addCallback(r -> System.out.println(r.getBody()), e -> ());
}
```

- 단점
    - 위의 예에서도 보듯이 스프링은 기본 톰캣 쓰레드 200개 중 계속 새로 생성해내는 방식으로 쓰레드를 점유하게 된다. 즉 논블로킹IO를 사용하지 않는다.
- 단점 보완
    - 그래서 non block io를 지원하는 방식으로 작업해야한다
        
        ```java
        Netty4ClientHttpRequestFactory factory 
        	= new Netty4ClientHttpRequestFactory(new NioEventLoopGroup(1))); // 논블로킹 IO쓰레드 1개만 할당
        AsyncRestTemplate art = new AsyncRestTemplate(factory); // 비동기 @MVC이므로 서블릿 쓰레드도 점유하지 않는다.
        ```
        

### 비동기 API호출의 조합과 결합은 어떻게?

- AsyncRestTemplate은 ListenableFuture로만 리턴
- @Async처럼 조합이 간편해지는 CompletableFuture로 리턴하면 안되나?
    - 리턴 오버로딩은 없음. CF는 LF의 서브타입도 아님
    - 스프링 이슈 트랙커의 답변 : 재주것 CompletableFuture로 만들어 써라
    
    ```java
    public <T> CompletableFuture<T> toCFuture(ListenableFuture<T> if) {
    	CompletableFuture<T> cf = new CompletableFuture<>();
    	if.addCallback((r) -> {
    		cf.complete(r);
    	}, (e) -> {
    	  cf.completeExceptionally(e);
    	});
    	return cf;
    }
    ```
    

## 결론

- 비동기작업과 API호출이 많은 @MVC앱이라면 아래방식을 종합적으로 고려해서 작성해야한다
    - AsyncRestTemplate + 논블로킹 IO라이브러리(netty, apache의 논블로킹io가 있음)
    - @Async와 적절한 TaskExecutor
    - ListenableFuture, CompletableFuture
- TaskExecutor(쓰레드풀)의 전략적 활용이 중요
    - 스프링의 모든 비동기 기술에는 ExecutorService의 세밀한 설정이 가능
    - CompletableFuture도 ExecutorService의 설계가 중요
    - 코드를 보고 각 작업이 어떤 쓰레드에서 어떤 방식으로 동작하는지, 그게 어떤 효과와 장점이 있는지 설명할 수 있어야 한다.
    - 벤치마킹과 모니터링 중요
- 비동기 스프링 기술을 사용하는 이유
    - IO가 많은 서버 앱에서 서버 자원의 효율적으로 사용해 성능을 높이려고(낮은 레이턴시 높은 처리율)
    - 서버 외부의 이벤트를 받아 처리하는 것과 같은 비동기 작업이 필요해서