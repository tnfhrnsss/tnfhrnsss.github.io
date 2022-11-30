---
layout: post
title: 더 자바, Java 8 수강 정리
date: 2022-11-30 20:17:00
last_modified_at : 2022-11-30 20:17:00
parent: Inflearn
grand_parent: Mooc
nav_exclude: true
tags: [java8]
---

* 소스 코드: [https://github.com/whiteship/java8](https://github.com/whiteship/java8)

* 강의주소 : [https://inf.run/qD8a](https://inf.run/qD8a)

## 수강 후기

- 몰랐던 내용도 배울 수 있었습니다
- java 문서를 제대로 봐야겠다고 다짐!

## 함수형 인터페이스

- *`추상형 메서드가 하나만 있으면 그게 함수형 인터페이스다`*
    - stiatic, default 메서드가 있어도 추상 메서드가 하나이기만 하면 함수형 인터페이스다.
    - 함수형 인터페이스를 정의하려면 `@FunctionalInterface`를 선언한다.
    - 멱등성
        - 입력받은 값이 동일하다면 결과값도 동일해야한다.
        - 그렇지 못한 경우
            - 외부에 있는 변수값을 변경하려 하는 경우
- 인터페이스임에도 static메서드를 정의할 수 있다 (public은 생략가능)
- default메서드도 정의할 수 있다.
- 인터페이스 상세 구현의 변화
    - 기존에는 익명 내부 클래스를 선언해서 구현했어야 했다.
    - 자바8부터는 메서드가 하나인 경우에는 람다표현식을 써서 줄여서 사용 가능해졌다 (= 인라인으로 표현가능)
        
        ```kotlin
        RunSomething rs = () -> System.out.println("Hello");
        ```
        
- api
    - function
        
        ```java
        Function<Integer, Integer> plus10 = (i) -> i + 10;
        ```
        
        - compose, andThen
            - 두개 이상 조합
            - 고차함수 higher-order function으로 함수가 함수를 매개변수로 받을 수 있고 리턴할 수 있다.
        
        ```java
        Function<Integer, Integer> plus10 = (i) -> i + 10;
        Function<Integer, Integer> multiply2 = (i) -> i * 2;
        
        plus10.compose(multiply2);
        
        // or
        plus10.andThen(multiply2).apply(2);
        ```
        
- BIFunction
- Cosumer
- Supplier
    - 인자 없이 받아올 값을 정의한다.
- Predicate
    - 인자 받아서 boolean리턴
- UnaryOperator
    - 인자와 리턴 타입이 일치할 때
    
    ```java
    UnaryOperator<Integer> plus10 = (i) -> i + 10;
    ```
    
- BinaryOperator
    - BiFunction의 특수한 형태로 세개의 타입이 모두 일치할 때, 하나만 입력하면 됨
- BiConsumer

# 람다 표현식

- (인자 리스트) -> {바디}
- 변수 캡쳐
    - 로컬 변수 캡처
        - final이거나 effective final 인 경우에만 참조할 수 있다.
            - effective final : final을 선언하지 않았지만, 사실상 final인 경우
        - 그렇지 않을 경우 concurrency 문제가 생길 수 있어서 컴파일가 방지한다.
    
    ```java
    private void run() {
        final int baseNumber = 10; // 이 변수가 사실상 final인 경우, final은 생략가능
        IntConsumer printInt = (i) -> System.out.println(i + baseNumber);
        
        printInt.accept(10);
    }
    ```
    
    - 익명 클래스 구현체와 달리 ‘쉐도윙’하지 않는다.
        - 익명 클래스는 새로 스콥을 만들지만, 람다는 람다를 감싸고 있는 스콥과 같다
        
        ```java
        private void run() {
            int baseNumber = 10;
            /*IntConsumer printInt = (i) -> System.out.println(i + baseNumber);
        
            printInt.accept(10);*/
        
            // 로컬 클래스
            class LocalClass {
                void printBaseNumber() {
                    int baseNumber = 11;    // 재정의를 하는 것을 위에 10인 변수를 가리는 것을 쉐도윙이라고 한다. 이것은 익명클래스도 마찬가지
                    System.out.println(baseNumber); // 11
                }
            }
        
            // 익명 클래스
            Consumer<Integer> integerConsumer = new Consumer<Integer>() {
                @Override
                public void accept(Integer integer) {
                    System.out.println(baseNumber); // 이건 10이지만
        
                }
            };
        
            // 익명 클래스
            Consumer<Integer> integerConsumer2 = new Consumer<Integer>() {
                @Override
                public void accept(Integer baseNumber) {    // 이렇게 인자를 통해 전달되면, 10인 baseNumber를 가리는 쉐도잉이 된다,
                    System.out.println(baseNumber); // 이건 10이 아닌 파라미터값이 된다,
        
                }
            };
        
            // 람다
            IntConsumer printInt = (i) -> {// 하지만 람다는  i대신에 baseNumber로 할 수 없다.
                System.out.println(i + baseNumber); // effective final이어야 한다.
            };
        
            printInt.accept(10);
        
            // 만약 아래처럼 바꾼다면, effective final이 아니기 때문에 람다식에서 참조 에러가 난다.
            //baseNumber++;
        }
        ```
        

## 메소드 레퍼런스

- 람다를 인라인으로 작성했던 부분을 메소드 참조로 변경
    
    ```java
    // 1
    UnaryOperator<String> hi = (s) -> "hi" + s; // 이렇게 가능하지만
    
    // 2
    Greeting greeting = new Greeting();
    UnaryOperator<String> hello = greeting::hello; // 클래스의 메소드 참조로도 구현가능
            
            
    ```
    
- 생성자도 참조할 수 있다.
    
    ```java
    public class Greeting {
        private String name;
    
        public Greeting() {
        }
    
        public Greeting(String name) {
            this.name = name;
        }
    
    		public String getName() {
            return name;
        }
    
        public String hello(String name) {
            return "hello " + name;
        }
    
        public static String hi(String name) {
            return "hi " + name;
        }
    }
    ```
    
    ```java
    // 이건 Greeting 인스턴스를 만드는 것이 아니다.
    Supplier<Greeting> newGreeting = Greeting::new; // 이건 인자가 없는 빈 생성자(Greeting()) 를 호출한 것 
    // 이때 인스턴스가 생성된다.
    newGreeting.get();  
    ```
    
    ```java
    Function<String, Greeting> greetingFunction = Greeting::new;  // 이건 String을 인자로 받는 public Greeting(String name) 생성자를 호출하는 것
    Greeting greeting = greetingFunction.apply("abc");
    
    ```
    
- 임의 객체의 인스턴스 메소드 참조
    
    ```java
    String[] nammes = {"aaa", "bbb", "ccc"};
    Arrays.sort(nammes, String::compareToIgnoreCase);   //  임의의 "aaa", "bbb", "ccc" 인스턴스들을 거쳐가며
    System.out.println(Arrays.toString(nammes));
    ```
    

# 인터페이스의 변화

## 인터페이스의 default 메소드와 static 메소드

- default 메소드
    - 이미 구현된 인터페이스에 공통 메서드를 또 추가할 경우 에러가 발생하는데 그때 default메서드를 사용하면 가능하다
    - Collection.java의 removeIf의 경우도 default메소드로 제공하고 있다.
    - 예
        
        ```java
        // interface class
        public interface Foo1 {
          void printName();
        
          default void printNameUpperCase() {
              System.out.println(getName().toUpperCase());
          }
        
          String getName();
        }
        
        // 구현 class
        public class DefaultFoo implements Foo1 {
          String name;
        
          public DefaultFoo(String name) {
              this.name = name;
          }
        
          @Override
          public void printName() {
              System.out.println(this.name);
          }
        
          @Override
          public String getName() {
              return this.name;
          }
        }
        
        // 호출 main메소드
        public static void main(String[] args) {
          Foo1 foo1 = new DefaultFoo("aaa");
          foo1.printName();
          foo1.printNameUpperCase();
        }
        ```
        
    - 하지만 default메소드를 구현한 부분이 항상 제대로 동작할 것이라는 보장이 안된다.
        - 구현하는 쪽에서 npe같은 오류가 발생하지 않도록 재정의를 한다.
        - Object가 제공하는 메서드는 default로 재정의할 수 없다.
            
            ```java
            default String toString() { --> 안됨
            }
            ```
            
    - 인터페이스를 상속받는 인터페이스에서 다시 추상 메소드로 변경할 수 있다
        
        ```java
        public interface Bar extends Foo1 {
            void printNameUpperCase();
        }
        ```
        
- static메소드
    - 해당 타입관련해서 유틸리티 메소드를 제공할때 인스턴스에 static메소드를 정의할 수 있다.

## 자바 8 API의 기본 메소드와 스태틱 메소드

- Iterable
    - foreach
    - spliterator
        - parallel하게 처리를 할 때 , 순서가 중요하지 않다면
        
        ```java
        List<String> name = new ArrayList<>();
        name.add("aaa");
        name.add("bbb");
        name.add("ccc");
        name.add("ddd");
        
        Spliterator<String> spliterator = name.spliterator();
        Spliterator<String> spliterator1 = spliterator.trySplit();
        while (spliterator.tryAdvance(System.out::println));
        while (spliterator1.tryAdvance(System.out::println));
        ```
        
- Collection
    - stream() / parallelStream()
        - 내부에 Spliterator를 써서 하고 있다.
    - removeIf(Predicate)
        - if조건에 해당하는 것을 배열에서 제거
    - spliterator()
- Comparator
    - functionalInterface다
    - sort할때 쓰임
    - api
        - reversed()
        - thenComparing()
        - static reverseOrder() / naturalOrder()
        - static nullsFirst() / nullsLast()
        - static comparing()
- default 메소드가 있기 때문에 같은 맥락에서 WebMvcConfigurer interface 와 이를 구현한 WebMvcConfigurerAdapter abstract클래스는 deprecated되었다.

# Stream

- 특징
    - Funtional in nature, 스트림이 처리하는 데이터 소스를 변경하지 않는다.
    - stream은 중개 operator와 종료 operator로 나눠진다.
    - 손쉽게 병렬처리할 수 있다.
- API
    - 중개 operator
        - 계속 이어나가는 것으로 stream으로 리턴된다.
        - filter, map, limit, skip, sorted…
        - 터미널 operator가 올때까지 리턴하지 않는다.
    - 터미널 operator
        - 종료시키는 것

# Optional

- 주의
    - 리턴값으로만 쓰기를 권장한다. (메소드 매개변수 타입, 맵의 키 타입, 인스턴스 필드
    타입으로 쓰지 말자.)
        - 매개변수타입으로 쓰면 null체크를 한번 더 해야하니까 번거롭게 될 뿐이다.
    - 프리미티브 타입용 Optional을 따로 있다. OptionalInt, OptionalLong,…
        
        ```java
        Optional.of(10); // 이건 박싱, 언박싱이 일어나서 성능에 안좋다
        OptionalInt.of(10); // 이렇게 사용하도록 하자
        ```
        
    - Optional을 반환해야하는데 null로 반환하지 말자. 대신 Optional.empty로 반환하자
    - Collection, Map, Stream Array, Optional은 Opiontal로 감싸지 말 것.
        - 두번 감싸게 되는 것
- api
    - Optional.isEmpty() → java11부터
    - Optional의 Optional은 flatMap을 써서 추출한다.

# Date와 Time

- Instant는 기계용, LocalDateTime은 휴먼용
- 기간 비교
    - Period - 휴먼용
        
        ```java
        Period.between();
        today.until();
        ```
        
    - Duration - 기계용
        
        ```java
        Duration.between()
        ```
        

# CompletableFuture

- Thread
    - 생성방식
        - 1안) Thread를 상속받아서 thread.run()
        - 2안) Thread안에 Runnable익명 클래스로 구현
        - 3안) 자바8 이후는 2안에서 람다로 변경해서 구현
- ExecutorService
    - shutdown은 graceful shutdown
    - shutdownNow()는 바로 terminal
    - Fork/Join
        - ExecutorService의 구현체
- Callable과 Future
    - Runnable과 유사하지만 리턴받을 수 있다는 차이가 있다
    - Future.get() 블로킹
- completableFuture
    - 비동기 인터페이스
    - 콜백도 다른 쓰레드로 줘서 실행시키도록 할 수 있다.
        
        ```java
        ExecutoreService executorService = Executors.newFixedThreadPool(4);
        CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
        	return "hello";
        }, executorService).thenRunAsync(() -> {
        	System.out.println(Thread.currentThread().getName());
        }, executorService);
        
        future.get();
        ```
        

# Annotation

- 어노테이션의 변화
    - 타입선언부에서도 사용할 수 있고,
        - @Target에 Type_Parameter, Type_use추가됨(9에선 module추가됨)
    - 중복해서 사용할 수 있다
        - @Repeatable

# 배열 Parallel 정렬

- Arrays.parallelSort()
    - 정렬할 때 Fork/Join 프레임워크를 사용해서 배열을 병렬로 정렬하는 기능을 제공한다.
    - 알고리즘의 효율은 같지만, 여러 쓰레드에 분산해서 하기 때문에 시간이 빠르다

# Metaspace

- Jvm의 여러 메모리 영역 중에 PermGen 메모리 영역이 없어지고 Metaspace영역이 생겼다.
- PermGen
    - permanent generation, 클래스 메타 데이터를 담는 곳
    - Heap영역에 속함
- Metaspace
    - 클래스 메타데이터를 담는 곳
    - Heap영역이 아니라, Native메모리 영역이다
    - 기본값으로 제한된 크기를 가지고 있지 않다.(필요한 만큼 계속 늘어난다.)
    - -XX:MaxMetaspaceSize=N
    - metaspace도 늘어나고 gc가 계속 일어난다는것은 어딘가 누수가 있는 것으로 모니터링해서 찾아내야함.