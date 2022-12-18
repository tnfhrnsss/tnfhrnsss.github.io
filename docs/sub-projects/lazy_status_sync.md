---
layout: post
title: lazy한 종료 상태 동기화 시도
date: 2022-12-18 14:00:00
last_modified_at : 2022-12-18 14:00:00
parent: Sub Projects
has_children: false
nav_exclude: true
---

## 배경

- 문제 상황
    - 여러 어플리케이션에서 전송하는 kafka 이벤트에서 비지니스 순서 보장이 되지 않는다.
        - 메시지 이벤트
        - 티켓 상태를 담은 이벤트
    - 비지니스 상태 정보를 담고 있는 이벤트와 사용자의 메시지 내용을 담고 있는 이벤트간 순서보장이 안됨(종료 메시지 이후 종료API를 호출해야하는 문제)
- 종료 상태 전파
    - 상담 비지니스 종료 후 고객에게 만족도 설문 메시지를 발송해야하는 케이스가 발생
    - 서드파티 채널 자체의 비지니스 상태를 갖고 있어서 상담 비지니스와의 상태 동기화가 필요함
        - 세션 종료 후 메시지 발송이 불가능하기 때문에, 메시지부터 모두 전송 후 세션 종료 처리가 필요함

## 설계

- 종료 상태 동기화에 대한 고민
    - 서드파티 채널 기준으로 보면, 비교적 종료 비지니스의 중요도가 떨어지는 상태로 판단됨
- 메시지와 티켓 상태 이벤트에 대한 의존성 제거
    - 모든 메시지는 티켓 상태에 상관없이 세션만 살아있으면 전송하게 한다.
    - 상태 또한 정보로써 갖고 있을 뿐 세션에 대한 제어를 하게 하지 않는다.
        - 대신, 서드파티 어플리케이션에서는 종료 이벤트를 받으면 티켓을 cold영역으로 이동시킨다
    - 종료 이벤트가 수신되면, 내부 프로세스를 마친 후 0.5 ~ 1초 내의 딜레이를 갖고 종료 API를 호출한다.

## 개발

- 종료된 티켓에 대한 정보 저장
    - ConcurrentHashMap에 종료된 티켓 정보를 저장
    - 티켓은 채널 정보 및 종료된 시점의 timestamp를 갖고 있다.
- 종료 API 호출
    - @Scheduled를 적용해서 일정 간격으로 일괄 호출하도록 한다.
    - clustering에 대한 검토
        - 대상 티켓을 ConcurrentHashMap에 담고 있기 때문에 서버별로 다르기도 하지만, batch가 동시에 각 서버에서 실행된다고 해도 문제가 되지 않는다. (중복해서 호출되어도 되는 API)
        - db를 조회한다거나 변경하는 부분은 없기 때문에 db lock에 대한 우려는 없음
- @Scheduled 적용시 검토한 사항
    - thread 재사용
        
        ```prolog
        @Bean
        public TaskScheduler taskScheduler() {
            ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
            scheduler.setPoolSize(8);
            scheduler.setThreadNamePrefix("Executor-CREMA-SCH-");
            scheduler.initialize();
            return scheduler;
        }
        ```
        
        - 동시에 여러개가 실행되는 것이 아니라 앞선 스케쥴링 종료 후 간격을 두고 호출하는 것이기에 pool size는 1로 둬도 된다.
    - @Async
        - 비동기로 여러개를 동시에 실행할 필요가 있다면 Async를 사용해도 되지만, 하나의 pool로 반복적으로 동작하기 때문에 async를 추가하지 않았다. (참고로 async의 default pool size는 8)
    - Scheduled 옵션
        
        ```java
        @Scheduled(
            fixedDelayString = "${crema.thirdparty.kakao.lazyEnd.fixedDelay:5000}",
            initialDelayString = "${crema.thirdparty.kakao.lazyEnd.initialDelay:10000}"
        )
        ```
        
        - 초기 지연 시간 10초
        - 이전 작업과의 간격 5초
        - log
            
            ```java
            14:22:28.856 DEBUG [,] [REMA-SCH-1] c.t.k.m.s.s.f.KakaoEndWaitingFlowService: 35 findAll
            14:22:33.862 DEBUG [,] [REMA-SCH-5] c.t.k.m.s.s.f.KakaoEndWaitingFlowService: 35 findAll
            ```
            

## 정리

- 기존에는 메시지간 순서를 지키기 위해, 메시지 내용으로 필터해서 하드코딩했던 부분이 있었는데, 그런 부분이 제거되면서 리팩토링이 될 수 있었다.
- 다른 어플리케이션의 비지니스에 의존도가 사라지게 되어, 상담 비지니스가 변경되어도 영향을 받지 않게 되었다.

## reference

[[Spring 레퍼런스] 26장 태스크(Task) 실행과 스케줄링 :: Outsider's Dev Story](https://blog.outsider.ne.kr/1066)

[Spring TaskExecutor와 TaskScheduler](https://m.blog.naver.com/estern/110135370787)