---
layout: post
title: 기타
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
has_children: false
nav_order: 4
parent: Home
# parent: Java
# grand_parent: Clipping
---

# 오버로딩 vs 가변 인수

1. java9의 컬렉션 API 중 List 인터페이스에서 List.of의 오버로드 메소드들이 존재하는 이유

→ 오버로드 말고 static <E> List<E> of(E... elements) 라고 선언할 수 있었음에도 사용하지 않은 이유

→ 내부적으로 가변 인수는 추가배열을 할당해서 리스트로 감싼다. 

→ 이렇게 되면, 가변인수로 받은 배열을 할당 → 초기화 → gc까지 실행해야하는 비용이 필요하게 된다.

비슷한 예로 Set.of, Map.of 도 마찬가지

# 객체지향

## :: 디미터의 법칙 (Law of Demeter)

디미터 법칙은 객체 O의 메소드 m은 다음의 객체들의 타입의 메소드만 호출해야 한다는 법칙이다.

1. O 객체 자신의 메소드들. (O itself)

2. m의 파라미터로 넘어온 객체들의 메소드들.(M's parameters)

3. m 안에서 생성 되거나 초기화된 객체의 메소드들.(Any objects created/instantiated within M)

4. O객체의 직접 소유하는 객체의 메소드들.(O's direct component objects)

5.O객체의 m에서 접근이 가능한 전역변수의 메소드들.(A global variable, accessible by O, in the scope of M)

```java
class Demeter {
    private A a;

    private int func() { return 0; }

    public void example(B b) {
        C c = new C();
       int f = func(); // 1번의 경우
        b.invert(); // 2번의 경우
       a = new A();
        a.setActive(); // 3번의 경우
       c.print(); // 4번의 경우
 
       // Static으로 설정된 Setting 변수가 있다고 가정할때.
        string UserID = GlobalValues.Setting.getUserID(); //5의 경우
    }
}
```

위의 종류에 해당하는 객체의 메소드들만 호출을 해야한다. 그렇지 않으면 유지보수 측면에서 문제점들이 발생한다. 아래의 예를 보면,

objectA.getObjectB().doSomething();

objectA.getObjectB().getObjectC().doSomething();

출처:

[https://hongjinhyeon.tistory.com/138](https://hongjinhyeon.tistory.com/138)