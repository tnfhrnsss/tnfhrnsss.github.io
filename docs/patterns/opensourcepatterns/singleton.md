---
layout: post
title: Singleton Pattern
date: 2023-03-02 23:11:45
last_modified_at : 2023-03-04 23:44:00
parent: opensourcepatterns
has_children: false
nav_exclude: true
---

- 어플리케이션 당 존재하는 객체의 인스턴스를 오직 한개만 보장하는 매커니즘으로 클래스 인스턴스를 하나만 만들고, 그 인스턴스로의 전역 접근을 제공한다. 
- 이 패턴은 공유 리소스를 관리할 때와 로깅과 같은 cross-cutting서비스를 제공할 때 유용하다.

# 1. usage
- 하나만 존재해도 잘 돌아가는 객체들을 대상으로 적용하는 패턴으로 인스턴스가 2개 이상이 되면 리소스를 잡아먹는다거나 결과에 일관성이 없다는 등의 문제가 생길 수 있다.
- 전역변수와의 차이점 : 전역변수는 어플리케이션이 시작될 때 객체가 생성되게 되는데 사용전에 이미 자원을 차지하게 되어 불필요한 객체 생성이 될 수 있다.
- 단점
  * 클래스 로더가 여러개라면 싱글턴 사용시, 클래스 로더를 직접 지정해서 여러번 로딩되지 않도록 해야한다.
  * 리플렉션, 직렬화, 역직렬호하도 싱글턴에서 문제가 된다.

# 2. singleton beans

- 일반적으로 싱글톤은 어플리케이션에 전역에 유일한 것이다. 하지만 스프링은 아니다. 이러한 제약은 느슨해졌고 대신에 스프링은 application이 아닌 spring Ioc 컨테이너당 객체 한개로 싱글톤을 규정하고 있다.
- 이 뜻은 스프링은 application context당 오직 한개의 bean만 생성한다
- 어플리케이션에는 여러개의 Spring 컨테이너를 존재할 수 있기 때문에 Spring의 접근 방식은 싱글톤의 엄격한 정의와 다르다.
- 따라서 컨테이너가 여러 개인 경우, 동일한 클래스의 여러 객체가 단일 애플리케이션에 존재할 수 있다.
- 디폴트는 Spring은 모든 bean들을 싱글톤으로 생성한다.
    - @Scope 어노테이션을 사용해서 빈 스코프를 변경할 수 있다.

# 2. Autowired Singletons

예를 들어서, 한개의 어플리케이션 context에 두개의 controller를 생성하고 각각 같은 타입으로 빈을 주입한다고 하면, 콘솔에 찍어봤을때 같은 object Id를 반환하는 것을 알 수 있다.

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("expiredSessions")
public class ExpiredSessionResource {
    private final ExpiredSessionFlowService expiredSessionFlowService;

    @PutMapping
    public void expiredSession() {
        System.out.println(expiredSessionFlowService);
    }
}
```

여러번 호출해서 찍어보면 ExpiredSessionFlowService@41d2528f 으로 동일하게 출력되는 것을 알 수 있다.

```java
09:20:52.579 TRACE [3c8343fc9684497a,3c8343fc9684497a] [050-exec-1] o.h.type.descriptor.sql.BasicBinder     : 64 bind            binding parameter [1] as [BIGINT] - [1675210852345]
spectra.attic.talk.crema.thirdparty.kakao.kakao.service.flow.ExpiredSessionFlowService@41d2528f

09:20:59.570 TRACE [447f5b5ee5bdf2fe,447f5b5ee5bdf2fe] [050-exec-2] o.h.type.descriptor.sql.BasicBinder     : 64 bind            binding parameter [1] as [BIGINT] - [1675210859549]
spectra.attic.talk.crema.thirdparty.kakao.kakao.service.flow.ExpiredSessionFlowService@41d2528f
```