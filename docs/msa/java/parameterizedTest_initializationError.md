---
layout: post
title: @ParameterizedTest와 initializationError
date: 2023-05-16 08:01:00
last_modified_at : 2023-05-16 08:01:00
parent: Java
grand_parent: Msa
has_children: false
nav_exclude: true
---

# Why initializationError?

### code miss

- ArgumentSource 클래스의 이름이나 패키지가 잘못되어 있거나, 클래스를 찾을 수 없는 경우
- ArgumentSource에서 반환하는 Arguments의 개수가 0개인 경우
- ArgumentSource에서 반환하는 Arguments에 null 값이 포함된 경우

### intellij 버그

- 위에 해당하지 않는데, annotation을 바꿔봐도, 디버깅을 태워봐도 잘 되는데 꼭 해당 메서드만 run하면 initializationError가 발생했다.([https://stackoverflow.com/a/49094543/14257397](https://stackoverflow.com/a/49094543/14257397))
- 프로젝트를 clean install 하면 잘 된다. 개별 메소드를 돌리면 꼭 마지막에 initializationError가 발생한다.

### Junit 버그

테스트 클래스에서 @ParameterizedTest 메소드 하나만 있으면 initializationError가 발생하는 경우

- 이 클래스에 @Test 메서드를 하나 추가하면 발생하지 않는다…
- chatgpt에 물어봤다.
    - 즉 Junit 버그

```
이러한 상황은 간혹 발생하는데, 이는 JUnit의 버그로 알려져 있습니다. 
JUnit은 테스트 클래스 내에 적어도 하나의 테스트 메서드가 있어야 한다는 것을 
요구합니다. 

@ParameterizedTest는 사실상 매개 변수화된 테스트 메서드이므로, 
JUnit은 해당 테스트 클래스에 테스트 메서드가 없다고 판단하고 
initializationError를 발생시킵니다.

하지만, @Test 어노테이션이 적용된 메서드가 하나라도 있으면 
JUnit은 테스트 클래스 내에 적어도 하나의 테스트 메서드가 있으므로 
initializationError를 발생시키지 않습니다. 

그래서 @ParameterizedTest와 함께 @Test 어노테이션이 적용된 메서드를 추가하면 
문제가 해결됩니다. 

다만, 이는 JUnit의 버그이므로 해결 방법이 아닌 임시적인 해결책입니다.
```

---

# Note

- @`ParameterizedTest`를 사용하는 경우, @Test 어노테이션은 필요하지 않다.
- @`ParameterizedTest`를 사용하는 경우 , 반드시 @MethodSource나 @ArgumentsSource와 같은 어노테이션을 활용해서 agument정의가 필요하다.

- **`@ParameterizedTest`**를 사용하는 경우, 각각의 테스트마다 매번 **`MockMvc`** 객체를 새로 만들어야 합니다. 이를 위해서는 **`@BeforeEach`** 또는 **`@Before`** 메소드를 사용하여 테스트 실행 전 매번 **`MockMvc`** 객체를 생성하고 초기화해주어야 합니다.

## Use @BeforeEach or @Before

- 만약 해당 테스트 메서드 내에서 전역 변수를 사용하는 코드가 있다면, 메서드 내에서 객체를 새로 만들어 줘야한다.
- 아니면  **`@BeforeEach`** 또는 **`@Before`** 메소드를 사용하여 테스트 실행 전 매번 객체를 생성하고 초기화해주어야 한다.
- 만약에 해당 작업을 하지않을 경우, 디버깅해보면 null값으로 되어있다.

## Can use `ArgumentsSource & ArgumentsProvider`

- 꼭 ArgumentsProvider를 통해서 작성하지 않아도 된다.
- 하지만 사용하지 않을 경우, 즉 `MethodSource`를 사용하는 경우에는 static 메서드로 작성해야한다. 안그러면 “cannot invoke non-static method “ 에러가 발생하게 된다.

## Or @MethodSource

- agument에 대한 정보가 꼭 동일 클래스나 패키지내에 있을 필요는 없다.
- 하지만 경로 및 메소드 명을 잘 기입해줘야한다.
- 만약에 아무값도 없으면 동일한 이름의 메소드를 찾는다.

```java
ex1) 동일 클래스
@MethodSource("provideArguments")

ex2) 동일 패키지 내에 다른 클래스
@MethodSource("parameterizedtest.StringParams#blankStrings")

ex3) 다른 패키지 및 다른 클래스
@MethodSource("message.sample.StringArgumentProviderA#provideArguments")
```

# Reference

- [https://velog.io/@ohzzi/junit5-parameterizedtest](https://velog.io/@ohzzi/junit5-parameterizedtest)
- [https://www.baeldung.com/parameterized-tests-junit-5](https://www.baeldung.com/parameterized-tests-junit-5)