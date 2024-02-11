---
layout: post
title: FeignClient with Multi-Product
description: FeignClient with Multi-Product
date: 2024-02-11 14:19:00
last_modified_at : 2024-02-11 14:19:00
parent: Feign
grand_parent: Msa
nav_exclude: true
tags: [feign, multi product]
---

# Question

- 업스트림 어플리케이션이 multi product구조로 변경되면서 동일한 API 호출을 해야할 때, FeignClient인터페이스 한개로 여러 번 호출이 가능한지…

# Thinking..

- spring-cloud-openfeign-core-3.1.6.jar 기준으로 FeignClient.class를 보면

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface FeignClient {
    @AliasFor("name")
    String value() default "";

    String contextId() default "";

    @AliasFor("value")
    String name() default "";

    /** @deprecated */
    @Deprecated
    String qualifier() default "";

    String[] qualifiers() default {};

    String url() default "";

    boolean decode404() default false;

    Class<?>[] configuration() default {};

    Class<?> fallback() default void.class;

    Class<?> fallbackFactory() default void.class;

    String path() default "";

    boolean primary() default true;
}
```

- url에 외부 주소를 설정해서 작성할 수 있고
- url이나 name을 동적으로 변경할 수 있고,
- 또한 yml로도 주입하는 등 여러 방법을 생각해볼 수 있지만
- 기본적으로 제공되는 변수만으로 eureka에 등록된 여러 어플리케이션을 멀티로 호출할 수 있지 않습니다.

# Conclusion

- FeignClient 인터페이스를 여러 개 생성해서 각 컨테이너별로 호출하는 구조로 작업을 했지만,
- 항상 멀티 프로덕트 컨테이너를 호출하는 것이 아니라, 조건에 따라 호출되는 컨테이너가 다르다면
- 여러 컨테이너에 대한 호출을 동적으로 처리하는 별도의 로직이나 구현이 필요합니다.
    - 중간에 분배기 역할을 하는 서비스를 둘 수도 있습니다.
- 즉, Feign 표준 기능으로는 멀티 프로덕트 어플리케이션 호출할 수 있는 속성을 갖고 있지 않습니다.