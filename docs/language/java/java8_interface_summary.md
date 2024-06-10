---
layout: post
title: java8+ interface 정리
date: 2024-06-10 21:41:00
last_modified_at : 2024-06-10 21:41:00
parent: Java
grand_parent: Language
has_children: false
nav_exclude: true
---

# Background

- [유튜브 강의](https://www.youtube.com/watch?v=T1BJzC9xb0g){:target="_blank"} 내용을 위주로 정리했습니다.
- 함수형 인터페이스에 대한 설명은 제외했습니다.
- 강의 내용 외에도 추가로 내용을 덧붙여서 정리했습니다. ✍

# 1. Java8 이후의 Interface 특징

- 인스턴스를 생성할 수 없다.
- 상수만 가질 수 있다
- body가 없는 추상 메서드를 가진다.
- jdk1.8 이상에서 디폴트 메소드 및 static 메서드를 선언할 수 있다.
- 그 밖에 함수형 인터페이스가 있다.

# 2. default method

## (1) 왜 default method가 생겼나?

- 이미 작성된 인터페이스에 기능을 추가할 때 구현체 클래스에 전부 override해야하는 경우가 생기는데, default 메서드로 대체하면 필요한 경우만 override하면 되기 때문

## (2) 특징

- interface에서 body를 가지는 메서드
- 반드시 override할 필요는 없다.
- 접근제어자는 public

# 3. static method

- java8이후부터 인터페이스에 정적메서드를 선언하고 인터페이스 타입을 통해 직접 호출될 수 있다.

```jsx
public interface AInterface {
    static void staticMethod() {
        System.out.println("Static method");
    }
}

// 구현체
AInterface.staticMethod();
```

## (1) 왜 static method가 생겼나?

- 인터페이스의 구현 클래스들이 공통으로 사용할 수 있는 유틸성 메서드를 작성하기 위함

# 4. Interface를 써야하는 이유

- 여러개의 인터페이스를 상속받을 수 있다.
    
    ```java
    public class ChildClass implements ParentA, ParentB {
    
    }
    ```
    
- 공통의 조상을 갖지 않는 여러 클래스의 상속을 맺을 수 있다.
- 이미 구현되어있는 클래스들의 하위호환성을 위해서

# 5. vs 추상클래스

| 인터페이스 | 추상클래스 |
| --- | --- |
| 관련이 없는 클래스끼리 관계를 맺어줄 때 | 밀접하게 관련된 클래스끼리 코드를 공유해야할 때 |
| 특정 데이터 타입의 동작을 지정하려고 하지만 해당 동작을 누가 구현하는지는 중요하지 않을 때 | 추상클래스의 하위 구현체 클래스들이 공통된 필드나 메서드를 많이 공유하고, 접근제어자가 public이 아닌 경우 |
| 다중 상속이 필요한 때 | non-static 혹은 non-final의 필드로 객체의 상태를 바꿔야하는 경우 |

# 6. Skeletal Implementation이란?

- 디자인 패턴 중 하나로 인터페이스와 추상클래스를 같이 사용해서 추상화를 고도화시키는 방법이다.
- 행위 중심 작성
- 예를 들어 collection의 AbstractList, AbstractMap, AbstractSet 등이 대표적 예시다.
    - 이들 추상 클래스를 상속받아서 필요한 메서드만 오버라이드하면 쉽게 커스터마이징이 가능하다.

## (1) 장점

- 코드 재사용
    - 동일 인터페이스를 구현하는 구현체가 여러개가 있고, 그 중 동일하게 동작하는 비지니스를 갖는 메서드가 존재한다면, 그 중복 메서드만 추상클래스로 구현한다. 그리고 구현클래스는 추상클래스와 인터페이스를 상속받는다면 중복코드가 사라지게 된다.
        - 하지만 리턴 후 후처리가 필요한 경우나, 시그니처가 다양할 경우는 어려울 수 있겠다.
    
    ```java
    // 인터페이스
    public interface AInterface {
    		void send();
    		
    		void close();
    }
    
    // 추상클래스
    public abstract class AAbstract implements AInterface {
    	@Override
    	public void send() {
    		// 전송
    	}
    }
    
    // 구현 클래스
    public class ChildClass extends AAbstract implements AInterface {
    	@Override
    	public void close() {
    		// 닫기
    	}
    }
    ```
    
    - 위의 케이스를 interface의 default메서드로도 가능할 수 있다.
        - 하지만 추상 골격 클래스는 private, protected가 가능하므로 Override한 메서드의 내부 메서드들을 클라이언트가 접근하지 못하도록 막을 수 있다.
        - 그러나 직접 interface를 구현하게 되면 접근 제어자가 모두 public이기 때문에 이와 같은 은닉이 힘들다.
        - 또한 디폴트 메서드에서는 하위 구현체 클래스에 대한 상태, 즉 필드에 대한 참조가 이루어질 수 없다.
- 유지보수성
    - 로직 변경이 필요한 경우 스켈레톤 클래스만 수정하면 되겠다.

# 7. Interface 사용시 주의점

- 디폴트 메서드는 런타임 오류를 일으킬 수 있으므로
    - 이미 구현된 인터페이스에 디폴트 메서드를 추가하는 것에 주의한다.
- 인터페이스 다중상속 구조에서 (시그니처가 동일한 ) 디폴트 메서드를 각각 Override한다면, 다이아몬드 문제가 일어나지 않나?
    - 이 경우 자바 컴파일러는 몇가지 규칙으로 해결하고 있다
        - 우선순위로 해결
            - 1순위: 구현하는 클래스나 슈퍼클래스
                
                ```java
                public interface A {
                	default void someMethod() {
                			System.out.println("Default in A");
                	}
                }
                
                public interface B extends A {
                	default void someMethod() {
                			System.out.println("Default in B");
                	}
                }
                
                public class C implements A {
                	@Override
                	public void someMethod() {
                		System.out.println("Default in C");
                	}
                }
                
                public class D extends C implements B {
                // -> 이 경우, super class인 C의 Default in C라고 출력된다.
                }
                ```
                
            - 2순위 : 상속받는 인터페이스
                
                ```java
                public interface A {
                	default void someMethod() {
                			System.out.println("Default in A");
                	}
                }
                
                public interface B extends A {
                	@Override
                	default void someMethod() {
                			System.out.println("Default in B");
                	}
                }
                
                public class C implements B {
                	// -> C의 someMethod를 호출하게 되면 B에서 재정의한 "Default in B"가 출력된다.
                }
                ```
                
            - 3순위 : 명시적 사용
                
                ```java
                public interface A {
                	default void someMethod() {
                			System.out.println("Default in A");
                	}
                }
                
                public interface B {
                	default void someMethod() {
                			System.out.println("Default in B");
                	}
                }
                
                public class C implements A, B {
                	public void someMethod() {
                		A.super().someMethod();
                	}
                }
                ```