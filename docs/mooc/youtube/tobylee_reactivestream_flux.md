---
layout: post
title: 토비의 봄 TV 스프링 리액티브 프로그래밍(14) - Flux의 특징과 활용방법 정리
date: 2023-03-07 21:05:00
last_modified_at : 2023-03-07 21:05:00
parent: Youtube
grand_parent: Mooc
nav_exclude: true
---

# 강의 주소

[토비의 봄 TV 스프링 리액티브 프로그래밍](https://www.youtube.com/live/bc4wTgA_2Xk?feature=share)

# 요점

- dependency
    - reactive-web

- Mono
    
    ```java
    @GetMapping("/event/{id}")
    Mono<Event> hello(@PathVariable long id) {
        return Mono.just(new Event(id, "event" + id));
    }
    ```
    
- Flux
    
    ```java
    @GetMapping("/events")
    Flux<Event> events() {
        return Flux.just(new Event(1L, "event1"), new Event(1L, "event1")); // 여러개의 인자
    }
    ```
    
    - 하지만 Mono로도 컬렉션 리턴이 가능하다
        
        ```java
        @GetMapping("/event/{id}")
        Mono<List<Event>> hello(@PathVariable long id) {
            List<Event> list = Arrays.asList(new Event(1L, "event1"), new Event(1L, "event1"));
            return Mono.just(list);
        }
        ```
        
- 컬렉션을 스트림으로 나눠서 보내고 싶다면, 미디어 타입을 스트림으로 지정한다.
    
    ```java
    @GetMapping(value = "/events_ex1", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
      Flux<Event> events_ex1() {
          return Flux.just(new Event(1L, "event1"), new Event(1L, "event1"));
      }
    ```
    
    - 약간의 시간차를 두고 리턴된다.
        
        ```prolog
        jhsim@jhsimui-MacBookPro toby_flux % curl localhost:8083/events_ex1                                                                                                                                                                               jhsim@jhsimui-MacBookPro toby_flux % curl localhost:8083/events_ex1
        data:{"id":1,"value":"event1"}
        
        data:{"id":1,"value":"event1"}
        ```
        
- 큰 데이터를 스트림을 나눠서 연속으로 내보낼 수 있다.
    
    ```java
    @GetMapping(value = "/events_ex2", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    	Flux<Event> events_ex2() {
            return Flux.fromStream(Stream.generate(() -> new Event(System.currentTimeMillis(), "value")))
                .delayElements(Duration.ofSeconds(1)) // 백그라운드 쓰레드를 별도로 만들어서 1초를 물고 있게된다. 메서드 자체가 blocking되는게 아님
                .take(10); // 10개의 request가 오면 cancel해서 끊는다.
        }
    }
    ```
    
- java의 Stream.generate대신에 Flux의 generate사용
    
    ```java
    @GetMapping(value = "/events_ex3", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    Flux<Event> events_ex3() {
        return Flux // sink는 데이터를 흘러내보내주는 것. next를 통해 다음 데이터를 가져온다.
                .<Event>generate(sink -> sink.next(new Event(System.currentTimeMillis(), "value")))  
                .delayElements(Duration.ofSeconds(1))
                .take(10);
    }
    ```
    
- 또는 초기값을 설정하고 알고리즘에 의해서 생성시키는 방법
    
    ```java
    @GetMapping(value = "/events_ex4", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    Flux<Event> events_ex4() {
      return Flux
              .<Event, Long>generate(() -> 1L, (id, sink) -> { // 초기값 1 설정하고 계속 증가시킴
                  sink.next(new Event(System.currentTimeMillis(), "value" + id));
                  return id + 1;
              })
              .delayElements(Duration.ofSeconds(1))
              .take(10);
    }
    ```
    
    - 동일한 zip방식
        
        ```java
        @GetMapping(value = "/events_ex6", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
        	Flux<Event> events_ex6() {
            Flux<String> es = Flux.generate(sink -> sink.next("Value"));
            Flux<Long> interval = Flux.interval(Duration.ofSeconds(1));
        
            return Flux.zip(es, interval).map(tu -> new Event(tu.getT2(), tu.getT1())).take(10);
        }
        ```