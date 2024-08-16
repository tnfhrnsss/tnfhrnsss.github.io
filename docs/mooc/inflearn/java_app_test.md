---
layout: post
title: 더 자바, 애플리케이션을 테스트하는 다양한 방법
date: 2022-12-24 22:11:00
last_modified_at : 2022-12-24 23:52:00
parent: Inflearn
grand_parent: Mooc
nav_exclude: true
---

강의자료 : [https://github.com/keesun/inflearn-the-java-test](https://github.com/keesun/inflearn-the-java-test){:target="_blank"}  

# 1. Junit5

- 모듈 구성
    - Platform: 테스트를 실행해주는 런처 제공 TestEngine API 제공
    - Jupiter: TestEngine 구현제로 Junit5를 제공
    - Vintage: Junit4와 3을 지원하는 TestEngine구현체
- 학습내용
    - junit5부터는 test메서드에 public을 생략해도 된다.
    - @BeforeAll, @AfterAll
        - static메서드로 작성해야한다. private은 안된다
        - 리턴타입은 void로 작성해야한다.
    - @BeforeEach, @AfterEach
        - static일 필요는 없다. void이기만 하면 된다.
- assertion
    - 메시지가 연산이 들어갈때 람다로 작성하자
    
    ```java
    assertEquals(StudyStatus.DRAFE, study.getSatus(), () -> "스터디를 처음 만든 상태다");
     // lamda로 작성하면 에러가 났을때만 실행되는 메시지. 문자열 연산이 크다고 하면 람다식으로 작성
     // 풀어쓰면 supplier로 된 부분
           new Supplier<String>() {
    
                @Override
                public String get() {
                    return "스터디를 처음 만든 상태다.";
                }
            };
    ```
    
- assertAll
    - 여러 assert를 람다로 해서 한번에 실행시킨다.
- 조건에 따라 테스트 실행하기
    - assumeTrue
        - false면 다음 테스트를 실행시키지 않는다.
        
        ```java
        assumeTrue("LOCAL", System.getenv("TEST_ENV"));
        ```
        
        ```java
        assumeTrue("LOCAL".equals(System.getenv("TEST_ENV")), () -> {
        
        });
        ```
        
        - annotation으로 @EnabledIfEnvironmentVariable(named=””, matches=””) 등으로 줄 수도 있다.
- custom annotation을 만들어서 적용할 수 있다.
- 인자값 넘길 수 있다.
    - @ValueSource, @CsvSource, @NullSource 등등
    - 인자값 조합시 ArgumentsAccessor 사용
    - `ArgumentsAggregator`로 인터페이스를 구현하는 클래스는 static으로 선언
- 테스트 인스턴스
    - test class는 메서드마다 인스턴스를 생성한다.
        - @TestInstance(TestInstance.Lifecycle.PER_CLASS) 을 선언하면  인스턴스를 하나만 만들게 되고 beforeAll, afterAll 등 도 static일 필요없게 된다.
- 테스트 순서
    - @TestMethodOrder(MethodOrderer.OrderAnnotation.class)
        - junit의 @Order를 써서 메서드마다 순서를 정할 수 있다.
- junit 설정파일을 둘 수 있다
    - junit-platform.properties를 resource로 추가하고, test resource directory로 설정해줘야 classpath에 올라간다.
- 확장 모델
    - junit4에서는 @RunWith(Runner), TestRule, MethodRule을 써서 확장하는 것이었고,
    - junit5에서는 Extension을 통해서 확장할 수 있다.
    - 확장 방법
        - @ExtendWith 선언적인 등록
        - @RegisterExtension 프로그래밍 등록
        - ServiceLoader이용해서 자동으로 등록

# 2. Mockito

- mockito-junit-jupiter는 mockito extension을 갖고 있는 라이브러리
- org.mockito.Mockito.mock
- @ExtendWith(MockitoExtension.class)
- 모든 Mock 객체의 행동
    - Null을 리턴한다
    - Primitive타입은 기본 Primitive값
    - 콜렉션은 비어있는 콜렉션
    - Voidㅁ서드는 예외를 던지지 않고 아무런 일도 발생하지 않는다.
- Mock 객체 확인
    
    ```java
    // memberService.notify 함수가 study객체로 한번 호출되었는지 확인
    verify(memberService, times(1)).notify(any());
    
    // memberservice.validate함수는 전혀 호출되지 않아야 한다.
    verify(memberService, never()).validate(any());
    ```
    
    - 순서에 맞게 호출되었는지 확인
        
        ```java
        InOrder inOrder = inOrder(memberService);
        inOrder.verify(memberService).notify(member);
        inOrder.verify(memberService).notify(study);
        ```
        
- Mockito BDD 스타일 API
    - BDD : 어플리케이션이 어떻게 “행동”해야하는지에 대한 공통된 이해를 구성하는 방법
    - 행동에 대한 스펙
        - Title
        - Narrative
            - As a / I want /so that
        - Acceptance criteria
            - Given/ When/ Then

# Testcontainers

- 단점 : db띄우는데 오래걸린다
- 장점 : 여러 db를 마련하지 않아도 테스트가능하기 때문에 production레벨로 테스트할 수 있다.

# JMeter

- 성능 측정 및 부하 테스트용 오픈소스 자바 어플리케이션
- 특징
    - cli 지원
- 주요 개념
    - Thread Group 한쓰레드 당 유저 한명
    - Sampler 어떤 유저가 해야 하는 액션
    - Listener 응답을 받았을 할 일(리포트, 검증, 그래프 그리기 등)
    - Configuration : Sampler 또는 Listener가 사용할 설정 값(쿠키, JDBC커넥션 등)
    - Assertion: 응답이 성공적인지 확인하는 방법(응답 코드, 본문 내용 등)
- 대체제: 게틀링, ngrinder
- apache bench test를 통해서도 api 요청 테스트가능
- 방법
    - jmeter를 어플리케이션 서버와 달리해야 리소스에 대한 점유를 분리할 수 있다.
    - 크롬의 blazemeter를 사용하면 jmeter파일로 생성시킬 수 있다. 그 파일을 cli통해서 실행할 수 있다
        
        ```java
        jmeter -n -t 설정파일 -i 리포트 파일
        ```
        

# 운영 이슈 테스트

## chaos monkey

- 소개
    - 카오스 엔지니어링 툴
        - 분산 시스템 환경에서 불확실성을 파악하고 해결 방안을 모색하는데 사용하는 툴
    - 운영 환경 불확실성의 예
        - 네트워크 지연
        - 서버 장애
        - 디스크 오작동
        - 메모리 누수
    - 스프링부트
        - 스프링 부트 어플리케이션을 망가트릴 수 있는 툴공격유형
        - 공격 대상(watcher)
            - restcontroller, controller, service, respository, component annotation이 붙은 것들
        - 공격 유형(Assaults)
            - 응답 지연(Latency Assault)
            - 예외 발생(Exception Assault)
            - 애플리케이션 종료(AppKiller Assault)
            - 메모리 누수(Memory Assault)

# 아키텍처 테스트

## Archunit

- 소개
    - 아키텍처란 패키지 구조, 클래스 관계, 레이어, 슬라이스간
    - 계층형 아키텍처, 도메인별로 모듈화된 아키텍처, 모놀리스, 양파형 아키텍처 등을 테스트