---
layout: post
title: 스프링 핵심 원리 - 고급편 수강 정리
date: 2022-11-30 20:20:00
last_modified_at : 2022-11-30 20:20:00
parent: Inflearn
grand_parent: Mooc
nav_exclude: true
tags: [spring]
---

## 좋은 설계란?

- 변경이 일어날 때 자연스럽게 드러난다.
- 비지니스 로직을 분리한다.
    - 변하는 로직과 변하지 않는 부분을 분리
- 단일 책임 원칙(SRP)을 구현한다.
    - 변경 지점을 하나로 모아서 변경에 쉽게 대처할 수 있는 구조를 만든다.

## 쓰레드 로컬 - ThreadLocal

- 동시성 문제
    - 싱글톤으로 등록된 겍체의 인스턴스 필드를 여러 쓰레드가 동시에 접근할 때 발생
    - 지역변수는 쓰레드마다 각각 다른 메모리 영역에 할당되기 때문에 동시성문제가 발생하지 않는다.
    - 인스턴스의 필드(주로 싱글톤에서) 또는 static같은 공용필드에 접근할 때 발생한다.
    - 동시성 문제는 값을 읽기만 하면 발생하지 않는다. 어디선가 값을 변경하기 때문에 발생
- 해결방법
    - ThreadLocal 사용
        - 주의 사항
            - 해당 쓰레드가 쓰레드 로컬을 모두 사용하고 나면 ThreadLocal.remove()를 호출해서 저장된 값을 모두 제거해줘야한다.
            - was풀을 사용하는 경우, 쓰레드 로컬 값이 남아있으면 문제가 발생한다.
                - was는 사용이 끝난 쓰레드 로컬을 쓰레드 풀에 반환한다. 쓰레드를 생성하는 비용은 비싸기 떄문에 쓰레드를 제거하지 않고, 보통 쓰레드 풀을 통해서 쓰레드를 재사용한다.

# 템플릿 메서드 패턴과 콜백 패턴

## 1. 템플릿 메서드 패턴

- 배경
    - 핵심 기능 vs 부가 기능
        - 핵심기능은 해당 객체가 제공하는 고유의 기능으로 리포지토리를 호출하는 코드 등이 해당한다.
        - 부가기능은 핵심 기능을 보조하기 위해 제공되는 기능으로 로그 추적 로직이나 트랜잭션 기능이 있다.
        - 두개는 섞여있으면 안된다.
    - 변하는 것과 변하지 않는 것을 분리한다
        - 좋은 설계는 변하는 것과 변하지 않는 것을 분리하는 것으로
        - 핵심 기능 부분은 변하고 부가 기능은 변하지 않는 부분이라 이 둘을 분리해서 모듈화해야한다.
- 정의
    - 목적
        - 알고리즘의 골격을 정의하고, 일부 단계를 하위 클래스로 연기한다.
        - 템플릿 메서드를 사용하면 하위 클래스가 알고리즘의 구조를 변경하지 않고도 알고리즘의 특정 단계를 재정의할 수 있다.
    - 단점
        - 상속에서 오는 단점을 그대로 가져간다. 즉 부모클래스와의 강결합이 문제
        - 부모클래스의 기능을 사용유무와 상관없이 강하게 의존하게 된다. (의존관계)
        - 별도의 클래스나 익명 내부 클래스를 생성해야 하는 부분도 복잡하다
- 다형성 구조 활용해서 템플릿 메서드 패턴 구현
    - 추상클래스 생성
        - 변하지 않는 로직을 정의한다
        - 추상메서드를 생성한다
        - 변하지 않는 로직내에서 추상메서드를 호출한다.
    - 자식클래스 생성
        - 추상변하는 로직을 상속받아 각각 상세 구현한다.
    - 실행 순서 : 추상클래스의 함수를 실행하면, 변하지 않는 로직을 태우고 중간에 변하는 로직은 각각 오버라이딩 된 자식클래스의 인스턴스가 실행된다.
    - 구조화를 잘해두면, 단일화의 원칙을 고수할 수 있다. 나중에 변경이 필요할때 변화가 적게 로직이 변경될 수 있는 장점이 있다.
- 익명 내부 클래스를 활용해서 템플릿 메서드 패턴 구현
    - 배경
        - 다형성 구조로 하게 되면 단점이 자식클래스를 계속 만들어야 한다는 것이다
        - 익명 내부 클래스를 사용하면, 객체 인스턴스를 생성하면서 동시에 생성할 클래스를 상속받은 자식 클래스를 정의할 수 있다. 이 클래스는 클래스 내부에 선언되기 때문에 익명 내부 클래스가 된다.
    
    ```java
    @Test
    public void templateMethod2() {
        AbstractTemplate template1 = new AbstractTemplate() {
    
            @Override
            protected void call() {
                log.info("비지니스 로직1 실행");
            }
        };
        log.info("클래스 이름1={}", template1.getClass()); // hello.advanced.trace.template.TemplateMethodTest$1
       template1.execute();
    
        AbstractTemplate template2 = new AbstractTemplate() {
    
            @Override
            protected void call() {
                log.info("비지니스 로직2 실행");
            }
        };
        log.info("클래스 이름2={}", template2.getClass()); // hello.advanced.trace.template.TemplateMethodTest$2
        template2.execute();
    }
    ```
    

## 2. 전략 패턴(Strategy pattern)

- 배경
    - 템플릿 메서드 패턴의 단점을 보완한다.
    - 스프링에서 의존관계 주입할 때 사용하는 방식이 전략 패턴이다.
- 방법
    - context 영역 : 변하지 않는 부분을 정의한 템플릿 역할을 하는 코드이다. 문맥은 변하지 않고 문맥속에서 전략을 변경한다고 생각하면 된다.
    - strategy 영역 : 인터페이스로 생성하며 변하는 부분에 대해 정의한다.
    - strategy인터페이스를 위임받아 상세 구현한다. (상속이 아닌 위임이다)
- 정의
    - 익명 내부 클래스 대신 람다로 strategy에 대한 상세 구현이 가능하지만, 이건 strategy 인터페이스의 메서드가 1개 일때만 가능하다
    - 단점
        - 전략을 변경하기 쉽지 않다.
        - 동시성 이슈가 발생할 수 있다.
- 구현방식
    - 전략을 필드로 선언해서 사용하는 방법이 있고
    - 전략을 파라미터로 전달받는 방식이 있다
        - 원하는 시점에 전략을 호출할 수 있다는 장점 - 유연성 확보

## 3. 템플릿 콜백 패턴

- 정의
    - 자바에서의 콜백
        - 실행 가능한 코드를 인수로 넘기려면 객체가 필요하다. 람다를 사용해서 한다.
        - 자바8 이전에는 보통 하나의 메서드를 가진 인터페이스를 구현하고, 익명 내부 클래스를 사용했다.
    - 스프링에서 전략을 파라미터로 받는 방식의 전략 패턴을 템플릿 콜백 패턴이라 한다. 즉, 전략부분이 콜백으로 넘어온다고 보면 된다.

## 4. 정리

- 한계점
    - 아무리 분리해서 최적화를 해도 코드를 다 수정해야하는 이슈가 생긴다.

# 프록시 패턴과 데코레이터 패턴

- 정의
    - 프록시란
        - 클라이언트가 서버로 직접 요청하는 것이 아닌, 대리자를 통해 간접적으로 요청할 수 있다.
        - 특징
            - 권한에 따른 접근제어
            - 캐싱
            - 지연 로딩
                - 실제 쓸때 데이터를 호출하는 것
            - 부가기능 추가
                - 서버가 제공하는 기능에 더해서 부가기능을 수행한다. 요청값이나 응답값을 변경
                - 실행 시간 측정해서 추가 로그를 남기거나 하는 것
            - 프록시 체인
            - 대체 가능
                - 객체에서 프록시가 되려면 클라이언트는 서버에게 요청할 것인지 프록시에게 요청할 것인지 몰라야 한다 (런타임 객체 의존 관계)—> 즉 서버와 프록시는 같은 인터페이스를 사용해야한다.
                - 클라이언트가 사용하는 서버객체를 프록시 객체로 변경해도 클라이언트 코드를 변경하지 않고 동작할 수 있어야 한다. —> DI를 사용해서 대체 가능하다.(프록시는 대체 가능해야하기 때문에 di를 쓰면 유연성을 확보할 수 있다)
        - 대리자가 또 다른 대리자를 부를 수 있다 —> 프록시 체인이라고 한다.
        - 객체에서의 프록시 역할
        - 프록시 패턴 적용 전은 클래스간 의존 관계로 되어있다.
- GOF 디자인 패턴
    - 프록시 패턴과 데코레이터 패턴은 둘다 프록시를 사용하지만, 이 둘을 의도(intent)에 따라서 구분한다
        - 프록시 패턴 : 접근 제어가 목적
        - 데코레이터 패턴 : 새로운 기능 추가가 목적
            - 프록시로 부가기능을 추가하는 것을 데코레이터 패턴이라고 한다.
                - 예: 요청값이나, 응답값을 중간에 변형하거나, 실행 시간을 측정해서 추가 로그를 남기는 것 등등
            - 데코레이터는 혼자 존재할 수 없다. 꾸며줄 대상이 있어야한다. 즉 호출할 대상을 갖고 있어야 하는데, 데코레이터별로 그 대상이 있기때문에 그 부분이 중복되어있기 때문에 이걸 추상화해서 중복을 제거시킨다.
- 프록시 패턴 vs 데코레이터 패턴(decorator라는 추상 클래스가 있으면 데코레이터 패턴인가?)
    - 의도(intent)
        - 디자인 패턴에서 중요한 것은 겉모양이 아니라 패턴을 만든 의도가 중요하기 때문에 의도에 따라 패턴을 구분한다.
        - 프록시 패턴의 의도 : 다른 객체에 대한 접근을 제어하기 위해 대리자를 제공
        - 데코레이터 패턴의 의도: 객체에 추가 책임(기능)을 동적으로 추가하고 기능 확장을 위한 유연한 대안 제공
- 구현
    - 인터페이스 없는 프록시
        - 자바의 다형성은 인터페이스를 구현하든 아니면 클래스를 상속하든 상위 타입만 맞으면 다형성이 적용된다. 그래서 인터페이스 없이도 프록시를 만들 수 있다.
        - 제약
            - 부모 클래스의 생성자를 호출해야한다
            - 클래스에 final키워드 붙으면 상속이 불가능하다
            - 메서드에 final이 붙으면 해당 메서드를 오버라이딩 할 수 없다.
    - 인터페이스 기반 프록시 vs 클래스 기반 프록시
        - 클래스 기반 프록시는 해당 클래스에만 적용할 수 있다. 인터페이스 프록시는 인터페이스만 같으면 모든 곳에 적용할 수 있다.
    

# 동적 프록시 기술

## 리플렉션

- 주의
    - 가급적 사용하지 않는다. 런타임에 동작하기 때문에 컴파일 오류를 잡을 수 없다.
- 동적 프록시
    - jdk 동적 프록시는 인터페이스를 기반으로 만들어준다.
    - `InvocationHandler`인터페이스 구현
    - cglib를 활용하는 방법 - spring framework에 포함되어있다.

# 스프링이 지원하는 프록시

## 프록시 팩토리

- 흐름
    - 프록시 요청이 오면, jdk동적 프록시와 cglib중에 프록시 기술을 선택해서 프록시를 반환받아 처리한다.
    - 프록시 팩토리를 사용하면 Advice를 호출하는 전용 InvocationHandler, MethodInterceptor를 내부에서 사용한다.
        - Advice는 프록시에 적용하는 부가기능 로직이다.
        - Advice는 jdk동적 프록시가 제공하는 InvocationHandler와 cglib가 제공하는 MethodInterceptor의 개념과 유사하다. 이 두개의 개념을 추상화한 것으로 프록시 팩토리를 사용하려고 하면 이 두개 대신에 Advice를 사용하면 된다.
    - 인스턴스에 인터페이스가 있으면 jdk동적 프록시를 기본으로 사용하고, 인터페이스가 없고 구현클래스만 있다면 cglib를 통해서 동적 프록시를 생성한다.
    - 특정 조건에서만 동작하는 프록시를 적용하는 경우는 PointCut을 사용한다.
- 포인트컷, 어드바이스, 어드바이저
    - pointcut
        - 어디에 부가기능을 적용할지 말지를 결정하기 위한 필터링 적용이다
        - 주로 클래스와 메서드 이름으로 필터링 한다.
    - advice : 프록시가 호출하는 부가기능으로 쓰인다. 프록시 로직으로 포함됨
    - advisor : 하나의 pointcut과 하나의 advice를 합친 것을 말한다.
- pointcut
    - 메소드는 classfilter, methodmatcher 두개로 이뤄져있다. 둘다 true로 반환해야 advice를 사용할 수 있다.
- 정리
    - 최근 스프링 부트 AOP는 기본적으로 proxyTargetClass를 true로 설정해서 사용한다. 따라서 인터페이스가 있어도 항상 cglib를 사용해서 구체 클래스를 기본으로 프록시를 생성한다.
    - 프록시 팩토리에서 advisor는 필수다.
    - AOP 적용 수만큼 프록시가 생성하는 것이 아니다. proxy는 하나만 만들고 하나의 proxy에 여러개의 advisor를 적용하는 방식으로 구현한다. 스프링AOP는 target마다 하나의 proxy만 생성한다.

# 빈 후처리기

- BeanPostProcessor
    - postProcessBeforeInitialization : 객체 생성 이후에 @PostConstruct같은 초기화가 발생하기 전에 호출되는 포스트 프로세서이다
    - postProcessAfterInitialization : 객체 생성 이후에 @PostConstruct 같은 초기화가 발생한 다음에 호출되는 포스트 프로세서이다.
- 빈 후처리 기능
    - 객체 조작가능
- dependency에 `org.springframework.boot:spring-boot-starter-aop`를 추가하면 자동 프록시 생성기(AutoProxyCreator)가 활성화된다.
- AutoProxyCreator
    - 자동으로 프록시를 생성해주는 빈 후처리기
    - AnnotationAwareAspectJAUtoProxyCreator
    - proxy는 하나만 생성해서 여러 Advisor를 연결하도록 한다.

# @Aspect AOP

- 어노테이션 기반 프록시를 적용할 때 필요하다.
- advisor = pointcut + advice로직을 @Aspect기반 어드바이저 빌더 내부에 저장한다.
- 스프링 컨테이너가 올라갈 때 @Aspect어노테이션이 있는 컴포넌트를 모두 스캔한다.
- 횡단 관심사(cross-cutting concerns) : 로깅처럼 어플리케이션의 여러 기능들 사이에 여러 곳에 걸쳐있는 공통 관심사

# 스프링 AOP(Aspect-Oriented Programming) 개념

- 배경
    - 부가기능을 어플리케이션의 여러 군데에 적용해야할 때, 많은 반복이 예상되고 중복코드도 많이 생기고 변경사항에 대한 작업도 예상되니 모듈화가 필요하다. 그런데 부가 기능처럼 특정 로직을 애플리케이션 전반에 적용하는 문제는 일반적인 oop방식으로 해결이 어렵다.
    - aop는 oop를 대체하기 위한 것이 아니라 횡단 관심사를 깔끔하게 처리하기 어려운 oop의 부족한 부분을 보조하는 목적으로 개발되었다.
    - 스프링 aop는 프록시를 사용한다.
- Aspect
    - 핵심기능과 부가기능을 분리
    - advisor도 어드바이스(부가 기능)과 포인트컷(적용대상)을 가지고 있어서 개념상 하나의 aspect다
- 동작 방식
    - 컴파일 시점 - aspectJ
        - 컴파일 타임(위빙) : 위빙은 aspect와 실제코드를 연결해서 붙이는 것을 말한다.
        - 컴파일러가 class파일 만들 때 부가기능 로직을 추가할 수 있다
        - AspectJ가 제공하는 특별한 컴파일러를 사용해야한다.
    - 클래스 로딩 시점 - aspectJ
        - class파일을 jvm에 저장하기 전에 조작할 수 있도록 해서 적용한다. 서비스 올릴 때 java옵션을 통해서 하는 거라 번거롭고 잘 안쓴다.
    - 런타임 시점(프록시)
        - aop가 이 방식임
        - 런타임 시점은 컴파일도 다 끝나고 클래서 로더에 클래스도 다 올라가서 이미 자바가 실행되고 난 다음을 말한다.
        - 스프링 aop의 조인 포인트는 메서드 실행으로 제한된다.
        - 프록시는 메서드 오버라이딩 개념으로 동작한다. 따라서 생성자나 static 메서드, 필드 값 접근에는 프록시 개념이 적용될 수 없다.
- aop 적용 위치 (join point)
    - 생성자, 필드 값 접근, static 메서드 접근, 메서드 실행
- aop 용어 정리
    - join point
        - 추상적인 개념으로 AOP를 적용할 수 있는 모든 지점이라 생각하면 된다
        - 하지만 스프링 AOP는 프록시 방식을 사용하므로 조인 포인트는 항상 메소드 실행 지점으로 제한된다
    - pointcut
        - 조인 포인트 중에서 어드바이스가 적용될 위치를 선별하는 기능
        - 주로 AspectJ표현식을 사용해서 지정
        - 프록시를 사용하는 스프링 AOP는 메서드 실행지점만 포인트컷으로 선별가능
    - target
        - 어드바이스를 받는 객체, 포인트컷으로 결정
    - Advice
        - 부가 기능
        - 특정 조인 포인트에서 Aspect에 의해 취해지는 조치
        - 종류 : Around, Before, After와 같은 다양한 어드바이스가 있다
    - Aspect
        - 어드바이스 + 포인트컷을 모듈화한 것으로 @Aspect다
        - 여러 어드바이스와 포인트 컷이 함께 존재
    - Advisor
        - 하나의 어드바이스와 하나의 포인트컷으로 구성
        - 스프링 AOP에서만 사용하는 용어
    - Weaving
        - 포인트컷으로 결정한 타겟의 조인 포인트에 어드바이스를 적용하는 것
        - 위빙을 통해 핵심 기능 코드에 영향을 주지 않고 부가 기능을 추가할 수 있다.
    - AOP 프록시
        - AOP 기능을 구현하기 위해 만든 프록시 객체, 스프링에서 AOP프록시는 JDK동적 프록시 또는 CGLIB프록시이다.

# 스프링 AOP구현

- pointcut 분리
    - pointcut을 나눠서 정의할 수 있다.
    - 메서드 이름 + 파라미터 = pointcut signature라고 한다. 아래 예시는 allOrder가 pointcut signature가 된다.
    - 메서드 반환타입은 void여야한다
    - 코드 내용은 비워둔다
    - public으로 하면 다른 aspect에서도 사용할 수 있다.
    - 다른 클래스에 있는 외부 advise에서도 pointcut을 사용할 수 있다.
        
        ```java
        @Pointcut("execution(* hello.aop.order..*(..))")
            private void allOrder() { // pointcut signature
        }
        
        @Around("allOrder()")
        public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
            log.info("[log] {}", joinPoint.getSignature());
            return joinPoint.proceed();
        }
        ```
        

- pointcut 참조 - pointcut을 외부에 모아놓고 사용하는 방법
    - 다 모은 pointcut 생성
    - 참조 클래스에서 아래처럼 arount정의
        
        ```java
        @Around("hello.aop.order.aop.Pointcuts.allOrder()")
        public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
            log.info("[log] {}", joinPoint.getSignature());
            return joinPoint.proceed();
        }
        ```
        
- Advise 순서
    - 기본적으로 순서를 보장하지 않는다.
    - 순서대로 적용하고 싶다면, @Aspect단위로 @Order를 적용해야한다. 단 advise단위가 아니라 class단위로 적용된다.
        - 내부클래스를 정의해서 @Aspect를 나누고 @Ordere로 순서지정해주면 된다.
- Advise 종류
    - @Around
        - 메서드 전후로 호출하는 가장 강력한 어드바이스로 조인 포인트 실행 여부 선택, 반환값 변환, 예외변환 등 가능하다
        - 이것만 있으면 다른 before, after, 등등 필요없다
        - joinPoint.proceed()호출 여부 선택
        - 전달값 변환
        - 반환 값 변환
        - 예외 변환
        - 트랜잭션 처럼 모두 들어가는 구문에 대해 처리가능
        - proceed()를 여러번 실행할 수도 있음
    - @After
        - finally역할과 비슷하다
- joinPoint 인터페이스의 주요기능
    - getArgs() : 메서드 인수를 반환
    - getThis()  : 프록시 객체를 반환
    - getTarget() : 대상 객체를 반환
    - getSignature() : 조인되는 메서드에 대한 설명을 반환
    - toString : 조인되는 방법에 대한 유용한 설명
- 순서
    - 스프링은 5.2.7버전부터 동일한 @Aspect안에서 동일한 조인포인트의 우선순위를 정했다.
    - 실행 순서 : @Around, @Before, @After, @AfterReturning, @AfterThrowing
        - 실행 순서와 리턴 순서는 반대가 된다.
        - @Aspect안에 동일한 종류의 어드바이스가 2개 있으면 순서가 보장되지 않는다. 이럴 땐 @Aspect를 분리하고 @Order를 적용하자

# 포인트 컷

- 지시자
    - execution과 같은 지시자를 줄여서 pcd(pointcut designator)라 한다
    - within : 특정 타입 내의 조인 포인트를 매칭한다.
    - args
    - this
    - target
    - @taget
    - @within
    - @annotation
    - @args
    - bean
- execution
    - 문법
        - execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)
            - ?는 생략가능
            - assert로 확인가능
                
                ```java
                @Test
                void exactMatch() {
                    pointcut.setExpression("execution(public String hello.aop.member.MemberServiceImpl.hello(String))");
                    Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
                }
                ```
                
            - hello.aop.member.*(1).*(2) 에서 (1)은 타입 (2)는 메서드 이름
            - 패키지에서 . 과 ..의 차이는
                - .은 정확하게 해당 위치의 패키지
                - .. 은 해당 위치의 패키지와 그 하위 패키지도 포함
            - 타입매칭
                - 상속받은 부모로도 매칭 허용된다.
            - execution의 args
                - match all
                    
                    ```java
                    execution(* *(..)))
                    ```
                    
                - String 타입으로 시작, 숫자와 무관하게 모든 파라미터, 모든 타입 허용
                    - (String) or (String, Xxx) or (String, Xxx, Xxx) 다 허용
                        
                        ```java
                        execution(* *(String, ..)))
                        ```
                        
                - (*, *) : 정확하게 두개의 파라미터만 허용하되, 단 모든 타입 가능
                - () : 파라미터가 없어야 함
        - within
            - execution이랑 비슷한데, 정확하게 타입 매칭하는 것
            - 단, 표현식에 부모타입(인터페이스 등)을 지정하면 안된다. 이점이 execution이랑 다르다.
            - 하나만 지정할 수 있기때문에 거의 사용안한다고 함
        - args
- @target vs @within
    - @target은 인스턴스의 모든 메서드를 조인 포인트로 적용한다. 즉 부모 클래스의 메서드까지 다 적용대상이고
    - @within은 해당 타입 내에 있는 메서드만 조인포인트로 적용한다, 자기자신의 클래스에 정의된 메서드만 적용한다. 즉 부모까지 올라가지 않음
    - 두개다 파라미터 바인딩으로 사용될 수 있다.
- bean
    - aspectj에도 없는 스프링만의 지시자이다
    - bean으로 맵핑하는 것
    
    ```java
    @Autowired
    private OrderService orderService;
    
    @Around("bean(orderService) || bean(*Repository)")
    ```
    
- 주의
    - args, @target, @args는 단독으로 사용하면 안된다. 이런 지시자가 있으면 모든 스프링빈에 대해서 어플리케이션 로딩시점에 aop가 적용되게 된다. 문제는 모든 스프링빈에 aop프록시를 적용하려고 하면 스프링이 내부에서 사용하는 빈 중에는 final로 지정된 빈들도 있기 때문에 오류가 발생할 수 있다. 이러한 표현식은 최대한 프록시 적용 대상을 축소하는 표현식과 함께 사용해야 한다. → 예를들어 @target은 execution과 같이 쓴다.
- 매개변수 전달
    - this vs target
        - this는 프록시 객체로 스프링 빈 객체를 대상으로 하는 조인 포인트
        - target은 실제 객체 대상을 말한다.
        - 두개는 프록시 생성 방식에 따른 차이

## 실무 주의사항

- 프록시와 내부 호출
    - 프록시 호출이 아니면 내부호출시 aop가 안걸리는 점이 문제
        - 1안
            - 자기 자신을 의존관계에 주입받는 것.  setter로 주입받고 this가 아니라 자기자신을 호출하는 것
            
            ```java
            public class CallServiceV1 {
                private CallServiceV1 callServiceV1;
                
                @Autowired
                public void setCallServiceV1(CallServiceV1 callServiceV1) {
                    this.callServiceV1 = callServiceV1;
                }
                
                public void external() {
                    log.info("call external");
                    callServiceV1.internal(); // this가 아닌
                }
            
                public void internal() {
                    log.info("call internal");
                }
            }
            ```
            
            - 단점 - 자기자신을 생성하면서 주입해야하기 때문에 주입할 때 실패가 날 수 있다
        - 2안
            - 1안 단점을 보완하는 방법으로 지연조회를 사용하는 것
            - *`ApplicationContext` 나* `ObjectProvider`를 사용
            
            ```java
            public class CallServiceV2 {
              //private final ApplicationContext applicationContext;
            
              private final ObjectProvider<CallServiceV2> callServiceV2ObjectProvider;
            
              /*public CallServiceV2(ApplicationContext applicationContext) {
                  this.applicationContext = applicationContext;
              }
              */
            
              public CallServiceV2(ObjectProvider<CallServiceV2> callServiceV2ObjectProvider) {
                  this.callServiceV2ObjectProvider = callServiceV2ObjectProvider;
              }
            
              public void external() {
                  log.info("call external");
                //  CallServiceV2 callServiceV2 = applicationContext.getBean(CallServiceV2.class);
                  CallServiceV2 callServiceV2 = callServiceV2ObjectProvider.getObject();
                  callServiceV2.internal();
              }
            
              public void internal() {
                  log.info("call internal");
              }
            }
            ```
            
        - 3안
            - 구조를 변경해서, 별도의 클래스로 내부 메서드를 옮기는 방법