---
layout: post
title: Factory Method Pattern
date: 2023-03-02 23:11:45
last_modified_at : 2023-03-02 23:11:45
parent: opensourcepatterns
has_children: false
nav_exclude: true
---

- 팩토리 메소드 패턴에는 원하는 객체를 생성하기 위한 abstract 메소드가 선언된 factory 클래스가 있어야한다.
- 특정 도메인에 기반한 각각의 다른 객체를 만들고자 할 때 사용한다. 예를 들어 car에 대한 객체가 필요한데, 해상 도메인과 항공 도메인 각각에 다른 객체인 배와 비행기를 만들 때 사용한다.
- 구현
    - 각 개체에 대한 팩토리 구현체를 만들고 구체적인 팩토리 메서드에서 원하는 개체를 반환할 수 있습니다.

# 1. Application Context

- 스프링은 DI 프레임워크에서 팩토리 메서드 패턴을 적용하고 있다
    - 즉, 스프링은 빈 컨테이너를 빈을 생성하는 팩토리처럼 다루고 있다
    - 게다가 스프링은 주요 함수를 추상화해서 빈컨테이너 안에 BeanFactory 인터페이스로 정의하고 있다.
        
        ```java
        public interface BeanFactory {
            getBean(Class<T> requiredType);
            getBean(Class<T> requiredType, Object... args);
            getBean(String name);
            // ...
        }
        ```
        
    - 이 getBean메서드들은 빈의 타입과 이름 등등으로 매칭되는 빈을 반환하는 팩토리 메서드인 셈이다.
    - Spring은 ApplicationContext 인터페이스로 BeanFactory를 확장해서 xml이나 annotation과 같은 일부 외부 구성을 한 bean 컨테이너를 올린다.
    - AnnotationConfigApplicationContext처럼 ApplicationContext 클래스를 implement해서 BeanFactory 인터페이스에서 상속된 다양한 팩토리 메소드를 통해 빈을 생성할 수 있다.
    

# 2. External Configuration

- 이 패턴은 외부 구성에 따라 애플리케이션의 동작을 완전히 변경할 수 있다.
- 만약에 어플리케이션의 autowire된 객체를 바꿔야한다면,

응용 프로그램에서 autowired 개체의 구현을 변경하려면 사용하는 ApplicationContext 구현을 조정할 수 있습니다.