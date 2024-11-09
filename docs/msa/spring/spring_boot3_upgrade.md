---
layout: post
title: spring boot 3 upgrade to-do list
date: 2024-09-19 16:27:00
last_modified_at : 2024-09-19 16:27:00
parent: Spring
grand_parent: Msa
nav_exclude: true
---

# spring boot 3 변경 사항 대응개발 준비
# 배경

- 곧? 🔜 적용 작업이 예상되기 때문에 차차 준비를 해나가려고 합니다. (저희는 2.7.x)
- 허니몬님 강의를 기준으로 대상 범위를 정하고
- 하나씩👷🏻 고쳐나가보면서 별도 포스트로 모아보겠습니다.

# [https://youtu.be/1WT6oxchM9M](https://youtu.be/1WT6oxchM9M){:target="_blank"}  

# 필수 요건

- java 17 ~ 20
- Jakarta EE 9 ~ 10
- Spring Framework 6.0 이상
- Maven 3(3.5+)
- Gradle 7.x(7.5+) and 8.x

# 변경 개요

## 1.Spring boot 3.x

### 1.1 Local variable Type interface

- usage
    - 변수 초기화
    
    ```bash
    var outputStream - new ByteArrayOutputStream();
    var hobbyList = new ArrayList<Hobby>();
    ```
    

### 1.2 switch 표현 방식 변경

- usage
    
    ```bash
    var numLetters = 0;
    numLetters = switch(day) {
    	case MONDAY, FRIDAY, SUNDAY -> 6;
    	case TUESDAY -> 7;
    	case THURSDAY, SATURDAY -> 8;
    	case WEDNESDAY -> 9;
    };
    ```
    

### 1.3 record

- data 와 비슷
- record 선언시
    - 접근자, 생성자, equals, hashCode, toString
- 용도
    - 순수한 데이터 이송 역할 의도
    - final 필드
- usage
    
    ```bash
    record Rectangle(double length, double width) {}
    ```
    

### 1.4 Pattern matching instanceof

- as-is 에서는 instanceof하고 나서 다시 캐스팅을 해야했는데
- to-be에서는 캐스팅 없이 할당된 변수 활용
    
    ```bash
    if (shape instanceof Rectangle r) {  // r에 저장되어있음
    	return 2 * 4.length()
    }
    ```
    

### 1.5 Java EE → Jakarta EE 9

- namespace 변경 : javax → jakarta
- java ee8에 대한 하위호환성 안됨
- 변경작업
    - intellij의 refactor → migrate package and classes → Java EE to Jakarta EE 를 통해 일괄 namespace 변경
- [https://www.samsungsds.com/kr/insights/java_jakarta.html](https://www.samsungsds.com/kr/insights/java_jakarta.html) 참고

### 1.6 기타

- @ConstructorBinding 변경 : 생성자가 2개이상인 경우 선언필요
- actuator
    - management.metrics.export.{product} → management.{product}.metrics.export로 변경
    - 민감데이터 기본 마스킹 처리 및 속성 설정
- 로깅 일자 형식 변경됨
- Apache Kafka 비동기 Acks 활성화
- jmx 종단점 노출 : health만 노출
- server.max-http-header-size: 웹컨테이너 일괄 적용
- Graceful Shutdown의 SmartLifeCycle 페이즈 변경
- Spring Data JDBC 자동구성 지원

---

## 2. Spring framework 6.0

- [What's-New-in-Spring-Framework-6.x](https://github.com/spring-projects/spring-framework/wiki/What%27s-New-in-Spring-Framework-6.x){:target="_blank"}  

### 2.1. 주요 특징

- Jakarta EE 9 ~ 10 지원
- JPA Hiberate ORM 6.1
    - jpa 2.x → 3.0 (jpa → jakarta persistence)
    - namespace 변경
- Jackson 2.14 Support
- Servlet 6.0
- Testing
    - HtmlUnit 2.64
    - Servlet API 6.0 기반의 서블릿 Mock 지원
- Jakarta Persistence 3.1.0
    - uuid 대체하는 tsid (time-sorted unique identifiers)
        - 18자리 넘버 = 13자리 스트링 - 13자리 epoch value
- RFC 7807
- PathPatternParser
    - 후행 슬래시 허용하지 않음 → 허용하려면 setUserTrainlingSlashMatch(true) 선언

### 2.2 General Core

- AOT(ahead of time) 을 통한 실행속도 향상
- GraalVM Native Image Support - 어플리케이션 최적화
- native configuration 죄적화 (maven, gradle 플러그인 제공)
    - bytecode, classpath, spring factories, properties 대상

### 2.3 HTTP interface client (@HttpExchange) 서비스 인터페이스

- feign 과 유사
- 프록시 생성해서 사용가능
- 사용하려면 webflux 추가해야함

### 2.4 Spring WebFlux

- PartEvent - Stream multipart form upload
- ResponseEntityExceptionalHandler - WebFlux exception
- Flux - non-streaming media types 반환
- Netty5 기반 Reactor Netty 2지원
- JDK HttpClient를 통합한 WebClient

# 업그레이드 순서

- java 17+ 버전 업 적용
- 네임스페이스변경
- 스프링 프레임워크 버전 업