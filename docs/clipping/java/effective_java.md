---
layout: post
title: Effective Java
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Home
# parent: Java
# grand_parent: Clipping
has_children: false
nav_order: 3
sitemap:
    changefreq: week
    priority: 1.0
---

# effective java 정리

1. **생성자 대신 정적 팩터리 메서드를 고려하라**
    
    ```java
    ex1) valueOf
    public static Boolean valueOf(boolean b) {
        return b ? Boolean.TRUE : Boolean.FALSE;
    }
    
    ex2) instance or getInstance
    Object newArray = Array.newInstance(classObject, arrayLen);  // instance 혹은 
    //getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
    
    ex3) getType
    FileStore fs = Files.getFileStore(path);   
    // getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다.
    ```
    
    (1) 장점
    
    - 이름을 가질 수 있다. : 생성자만으로는 반환될 객체의 특성을 제대로 설명하지 못한다. 여러개의 생성자만으로는 어떤 역할을 하는지 구분하기 어렵다.
    - 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다. : 생성 비용이 큰 객체가 자주 요청되는 상황이라면 성능을 높일 수 있다. 플라이웨이트 패턴도 비슷한 기법이라고 할 수 있다.
    - 인스턴스 통제가 가능하다. : 반복되는 요청에 같은 객체를 반환하는 식으로 언제 어느 인스턴스를 살아 있게 할지 철저히 통제할 수 있다. 즉, 싱글턴으로 만들수 있고 인스턴스화불가 상태로 만들 수도 있다. 예를 들어 인스턴스 통제를 통해 Enum타입을 인스턴스가 하나만 만들어짐을 보장한다.
    - 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다. : API를 만들때 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어서 API를 작게 유지할 수 있다.
    - 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다. :반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.  (예. EnumSet클래스에선 원소의 수에 따라 RegularEnumSet 또는 JumboEnumSet의 인스턴스를 반환)
    - 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. : service provider framework가 그런 경우인데, 클라이언트는 서비스 접근 api를 사용할 때 원하는 구현체의 조건을 명시할 수 있다. 조건이 없으면 기본 구현체를 반환하거나 다른 구현체를 반환한다.
    
    (2) 단점
    
    - 상속을 할 수 없다 : 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
    - 정적 팩터리 메서드는 프로그래머가 찾기 어렵다. : 생성자처럼 API 문서에 명확히 드러나지 않으니 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야한다.
    
    **2. 생성자에 매개변수가 많다면 빌더를 고려하라**
    
    [문제상황]
    
    매개변수가 많고 호출 경우가 다양해서 생성자로 설정을 하게 되면 보통 사용자가 설정하기 원치 않는 매개변수까지 설정해야하는 경우가 생기는데, 필드가 더 많아지게 되면 값의 의미가 무엇인지 헷갈릴 것이고 매개변수가 몇 개인지도 주의해서 세어보아야 할 것이다.
    
    ** 자바빈즈 패턴 :  선택 매개변수가 많을 때 활용할 수 있는데, 매개변수가 없는 생성자로 객체를 만든 후, setter메서드들을 호출해서 원하는 매개변수의 값을 설정하는 방식. 근데 이 자바빈즈 패턴은 일관성이 깨지고, 불변으로 만들수 없다. 디버깅도 어렵고 스레드 안전성도 보장하기 어려움
    
    [빌더패턴 활용]
    
    - 필수 매개변수만으로 생성자 혹은 정적 팩터리를 호출해서 빌더 객체를 얻는다.
    
    ```java
    public class NutritionFacts {
        private final int servingSize;
        private final int servings;
    
        public static class Builder {
           private final int servingSize;
           private final int servings;
    
           public Builder(int servingSize, int servings) {
              this.servingSize = servingSize;
              this.servings = servings;
           }
        }
        
        private NutritionFacts(Builder builder) {
            serviceSize = builder.servingSize;
            servings = builder.servings;
        }
    }
    ```
    
    ```java
    [caller]
    NutritionFacts cocaCola = new NutritionFacts.Builder(240 ,8)
      .calories(100).sodium(35).carbohydrate(27).build();
    ```
    
    - 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다. (피자 예시)
        - 각 계층의 클래스에 관련 빌더를 멤버로 정의하고
        - 추상 클래스는 추상 빌더를
        - 구현 클래스는 구현 빌더를 둔다.
    - 단점
        - 빌더 생성비용에 따라서 성능에 문제가 될 수 있다.
        - 매개변수가 4개 이상일 때, 점층적 생성자 패턴보다 활용
    
    3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
    
    - 생성자를 만들지 않으면 자동으로 기본생성자가 만들어지기때문에 인스턴스화가 되버린다. 이때 private생성자를 추가해서 인스턴스화를 막는다. —> 이 방식으로 상속도 막게 된다. (상위 클래스의 생성자에 접근못하므로)
    
    4. 인스턴스화를 막으려거든 private 생성자를 사용하라
    
    5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
    
    - 많은 클래스가 의존하는 자원일 경우, 많이들 정적 클래스로 선언해서 사용하는데, 사용하는 자원에 따라 동작이 달라진다면 정적 유틸리티 클래스나 싱글턴 방식은 맞지 않다.
    - 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식으로 해본다.
    
    ```java
    public class SpellChecker {
        private final Lexiton dictionary;
    
        public SpellCheck(Lexiton dictionary) {
            this.dictionary = Objects.requireNonNull(dictionary);
        }
    }
    ```
    

6. 불필요한 객체 생성을 피하라

- 생성자 대신 정적 팩터리 메서드를 만들면 불필요한 객체 생성을 피할 수 있다.

(ex. Boolean.valueOf(String) )

- 하지만 문자열을 검사하는 메소드의 경우는 비용이 드므로, 캐싱해서 재사용하도록 한다.

(ex. Pattern은 정규표현식에 해당하는 finite state machine을 만들기 때문에 인스턴스 생성 비용이 높다.  —> 이 경우는 Pattern 인스턴스를 클래스 초기화할 때 캐싱해두고 인스턴스를 재사용하면 된다.)

- 오토박싱의 경우도 비용이 든다. 박싱된 기본 타입보다 기본타입을 사용하고 오토박싱이 되지 않도록 한다.

7. finalizer와 cleaner사용을 피하라

- 언제 실행될지 예측할 수 없고, 실행될 때까지 열어둔다면 열수 있는 파일 개수에 영향이 가게된다.
- 수행여부조차 보장하지 않는다.
- 가비지 컬렉터의 효율을 떨어뜨린다

8. equals

- 동치성 검사는 언제 하는가, 즉 재정의가 필요할 때는?
    - 상위 클래스의 equals가 재정의되지 않았을 때, 주로 값 관련 클래스들이 그렇다
- equals를 재정의했으면 반드시 hashcode도 재정의해야한다.
    
    
1. toString을 재정의해야하는 이유
- Object의 기본 toString으로는 정보를 알수 없으므로, 사람이 알아볼수 있는 포맷을 갖춰서 반환하도록 해야한다.
- 쓸모없는 메시지로 표시되는 것보단, 알아볼수 있다면 디버깅에도 도움
- toString이 필요없는 경우
    - 정적 유틸리티 클래스
    - 열거타입 클래스 - 자바가 이미 제공하는 것으로 추운

1. 상속보다는 컴포지션을 사용하라
    - 상속 후 재정의로 인해 발생할 수 있는 오류를 방지해야한다.
    - 컴포지션 설계 - 재정의가 필요하면, 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private필드로 기존 클래스의 인스턴스를 참조하게끔 한다.
    - 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야한다.
    

21. 인터페이스는 구현하는 쪽을 생각해 설계하라

- 디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 디폴트 메서드를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다.
    - 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운 법이다.

1. 태그 달린 클래스보다는 클래스 계층구조를 활용하라
- 태그 달린 클래스란?
    - 클래스 중에 두가지 이상의 의미를 표현할 수 있으며, 그 중 표현하는 의미를 태그 값으로 알려주는 클래스
    - 열거 타입 선언, 태그 필드, switch문 등 코드가 섞여있는 클래스
    - 여러 구현이 혼합돼 있어서 가독성이 나쁘다.
    - 그 클래스에 새로운 의미를 추가할 때마다 모든 switch문에 코드를 추가해야하고
    - 인스턴스의 타입만으로는 나타내는 의미를 알 수 없다.

```java
// ex) 태그달린 클래스
class Figure {
	enum Shape { RECTANGLE, CIRCLE };

	final Shape shape;

	double radius;

	Figure(double radius) {
		shape = Shape.CIRCLE:
		this.radius = radius;
	}

	double area() {
		switch(shape) {
			case RECTANGLE;
				return length * width;
			default:
				throw new AssertionError(shape);
		}
	}
}
```

- 태그달린 클래스 → 클래스 계층 구조로 변경한다.
    - 계층구조의 root가 되는 추상 클래스를 정의
    - 태그값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.
    - 태그에 상관없이 동작이 동일한 메서드들을 일반 메서드로 선언한다.
    
    ```java
    abstract class Figure {
    	abstract double area();
    }
    
    class Circle extends Figure {
    	final double radius;
    
    	Circle(double radius) {
    		this.radius = radius;
    	}
    
    	@Override double area() {
    		return Math.PI * (radius * radius);
    	}
    }
    
    class Rectangle extends Figure {
    	final double length;
    	final double width;
    
    	@Override double area() {
    		return length * width;
    	}
    }
    ```
    

1. raw 타입은 사용하지 마라
- 로 타입을 쓰게되면 타입 안정성을 잃게 된다.
- 비한정적 와일드 카드 타입을 사용하라 - 로타입 컬렉션은 아무 원소나 넣을 수 있으니 타입불변식을 훼손할 수 있는데, Collection<?>은 null외에는 어떤 원소도 넣을 수 없다.
- 예외) class리터럴에는 raw타입을 써야한다. - 예를 들어, List.class,  String[].class. int.class는 허용하고 List<String>.class 와 List<?>.class는 허용하지 않는다.

1. 배열보다는 리스트를 사용하라
- 배열은 공변이다. 상위 타입과 하위 타입이 같이 변한다는 것. 반면 제네릭은 불공변이다.

1. 한정적 와일드카드를 사용해 API유연성을 높이라
- 펙스(PECS) 공식: producer-extends, consumer-super
    - 매개변수화 타입 T가 생산자(producer)라면 <? extends T>를 사용하고
    - 소비자(consumer)라면 <? super T>를 사용하라
    
1. 제네릭과 가변인수를 함께 쓸 때는 신중하라
- 제네릭과 varargs를 혼용하면 타입 안전성이 깨진다.

```java
static void dangerous(List<String>... stringLists) {
		List<Integer> intList = List.of(42);
		Object[] object = stringLists;
		objects[0] = intList;
			String s = stringLists[0].get(0);  // ClassCastException발생
}
```

- 제네릭 varargs배열 매개변수에 값을 저장하는 것은 안전하지 않다.

1. [타입 안전 이종 컨테이너를 고려하라](https://www.notion.so/a23a9161427c4dd8af9790bac3b4dc76)

1. int 상수 대신 열거 타입을 사용하라
- 열거 타입이 근본적으로 불변이지만, 모든 필드는 final이어야 한다.
- 필드를 public으로 선언해도 되지만, private으로 두고 별도의 public접근자 메소드를 두는게 낫다

```java
public enum Planet {
	EARTH(3.302e+23, 2.439e6);
	
	private final double mass;

	Planet(double mass) {
		this.mass = mass;
	}

	public double mass() {
		return mass;
	}
}
```

- 열거타입 클래스에 상수별로 다르게 동작해야 하는 경우, switch문 대신 상수별 메서드 구현을 사용하자
- 열거타입 상수 일부가 같은 동작을 공유한다면 [전락 열거 타입 패턴](https://www.notion.so/f72a372488114e78bccb555dd9ab5e7c)을 사용하자.

1. 비트 필드 대신 EnumSet을 사용하라.
2. ordinal 인덱싱 대신 EnumMap을 사용하라.

```java
Map<Plant.LifeCycle, Set<Planet>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
	plantsByLifeCycle.put(lc, new HashSet<>());
}

for (Plant p :garden) {
	plantsByLifeCycle.get(p.lifeCycle).add(p);
}
```

- EnumMap은 내부에서 배열을 사용하기 떄문에, ordinal 성능과 비등하다.
- 아니면 스트림을 사용하면, 코드를 더 간략하게 할 수 있다

```java
Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle));
Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle, () -> 
		new EnumMap<>(LifeCycle.class), toSet()));
```

1. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라.
- 열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다. (예: java.nio.file.LinkOption열거타입은 CopyOption, OpenOption인터페이스를 구현했다)

```java
// Operation.class
public interface Operation {
	double apply(double x, double y);
}

// BasicOperation.class
public enum BasicOperation implements Operation {
	PLUS("+") {
		public double apply(double x, double y) {return x + y;}
	};

	private final String symbol;

	BasicOperation(String symbol) {
		this.symbol = symbol;
	}

	@Override
	public String toString() {
		return symbol;
	}
}

// 확장가능 열거 타입 ExtendedOperation.class
public enum ExtendedOperation implements Operation {
	EXP("^") {
		public double apply(double x, double y) {
			return Math.pow(x, y);	
		}
	};

	private final String symbol;

	ExtendedOperation(String symbol) {
		this.symbol = symbol;
	}

	@Override
	public String toString() {
		return symbol;
	}
}
```

1. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라
- 아무 메서드도 담고 있지 않고, 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스를 마커 인터페이스라고 한다.  (예: Serializable)
- 마커 인터페이스가 마커 애너테이션 보다 나은 점
    - 마커 인터페이스는 이를 구현한 클래스의 인스턴스를 구분하는 타입으로 쓸 수 있다.
    - 적용 대상을 더 정밀하게 지정할 수 있다.
        - 애너테이션으로 하면 Element.Type에 한정되는 반면, 인터페이스는 확장하면 그 하위 타입으로 보장된다.

## :: 람다와 스트림

1. 익명 클래스보다는 람다를 사용하라

```java
[as-is]
Collections.sort(words, new Comparator<String>() {
	public int compare(String s1, String s2) {
		return Integer.compare(s1.length(), s2.length());
	}
});

[to-be]
Collectoins.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));

-->

Collection.sort(words, comparingInt(String::length));

-->

words.sort(comparingInt(String::length));
```

- 전략패턴처럼, 함수 객체를 사용하는 과거 객체 지향 디자인 패턴에는 익명 클래스명 충분했다.
- 자바8에서는 추상 메서드 하나짜리 인터페이스를 람다식으로 만들어 간략하게 사용할 수 있다.
    - 타입을 명시헤야 하는 경우를 제외하고는 람다의 모든 매개변수 타입은 생략하자
- 람다는 이름이 없고 문서화도 못한다.
    - 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야한다. (최대 3줄 안에 끝내는 것이 좋다)
- 람다 함수 객체가 자신을 참조해야할 때는 this 키워드가 바깥 인스턴스를 가리키기 때문에 쓸수 없다. 이 경우는 익명 클래스를 써야한다.
- 람다도 익명 클래스처럼 직렬화 형태가 구현별로 다를 수 있다. 따라서 람다를 직렬화하는 일은 극히 삼가야한다(익명 클래스의 인스턴스도 마찬가지). Comparator처럼 직렬화를 해야만한다면 private정적 중첩 클래스의 인스턴스를 사용하자

1. 람다보다는 메서드 참조를 사용하라
- 매개변수가 늘어날수록 메서드 참조로 제거할 수 있는 코드양도 늘어난다.

```java
[as-is]
map.merge(key, 1, (count, incr) -> count + incr);

[to-be]
map.merge(key, 1, Ingeger::sum);
```

- 람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다.
- 때로는 메서드 참조보다 람다가 더 간결할 때가 있다.
- 제네릭 함수 타입의 경우 람다로는 불가능하나 메서드 참조로 가능하다.

```java
interface G1 {
	<E extends Exception> Object m() throws E;
}

interface G2 {
	<F extends Exception> String m() throws Exception;
}

interface G extends G1, G2 {}

<F extends Exception> () -> String throws F
```

1. 표준 함수형 인터페이스를 사용하라
- 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자 - 계산이 많을 때 성능이 느려진다.

1. 스트림은 주의해서 사용하라
- 스트림 파이프라인은 지연 평가된다. 평가는 종단 연산이 호출될 때 이뤄지며 무한 스트림을 다룰 수 있게 해주는 열쇠이다.
- 람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.
- char용 스트림을 지원하지 않기 때문에, char값을 처리할 때는 스트림을 삼가는 편이 낫다.

1. 스트림에서는 부작용 없는 함수를 사용하라
- 스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다. 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.

```java
// 잘못된 스트림 사용
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
	words.forEach(word -> {   // forEach는 그저 스트림이 수행한 연산 결과를 보여주는 일만 해야한다.
		freq.merge(word.toLowerCase(), 1L, Long::sum); // freq라는 외부 상태를 수정하는 람다를 실행하는 문제
	});
}

// 올바른 스트림 사용
Map<String, Long> freq = 
try (Stream<String> words = new Scanner(file).tokens()) {
	freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

- forEach연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는데 쓰지 말자.
- 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 toMap을 사용할 수 있다

```java
// 문자열을 열거타입 상수에 맵핑한다.
private static final Map<String, Operation> stringtoEnum =
	Stream.of(values()).collect(
		toMap(Object::toString, e -> e));
```

```java
// 각 키와 해당 키의 특정 원소를 연관 짓는 맵을 생성하는 수집기
Map<ARtist, Album> topHits = albums.collect(
	toMap(Album::artist, a -> a, maxBy(comparing(Album::sales))));
```

```java
// 마지막에 쓴 값을 취하는 수집기
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

1. 반환 타입으로는 스트림보다 컬렉션이 낫다
- for-each로 스트림을 반복할 수 없는 이유는 Stream이 Iterable을 extand하지 않아서이다.
- 원소 시퀀스를 반환하는 공개API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다
    - Collection인터페이스는 Iterable의 하위 타입이고 stream메서드도 제공하니 반복과 스트림을 동시에 지원한다.
    - Array도 Array.asList와 Stream.of메서드로 반복과 스트림을 지원할 수 있다.

```java
public static <E> Stream<List<E>> of(List<E> list) {
	return IntStream.range(0, list.size())
				.mapToObj(start ->
					IntStream.rangeClosed(Start + 1, list.size())
						.mapToObj(end -> list.subList(start, end)))
				.flatMap(x -> x);
}
```

1. 스트림 병렬화는 주의해서 적용하라
- 환경이 아무디 좋아도 데이터 소스가 Stream.iterate거나 중간 연산으로 limit을 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.
- 대체로 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다
    - 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 스레드에 분배하기 좋다는 특징이 있다
    - 원소들을 순차적으로 실행할 때의 참조 지역성이 뛰어나다. 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 아주 중요한 요소로 작용한다.

## :: 메소드

1. 매개변수가 유효한지 검사하라.
- requireNonNull : null검사를 수동으로 하지 않아도 된다. 원하는 예외 메시지를 지정할 수 있다. 입력값을 그대로 반환도 된다.

```java
this.strategy = Objects.requireNonNull(strage, "전략");
```

- 자바9에서는 Object의 범위 검사 기능을 추가했다. : checkFromIndexSize, checkFromToIndex, checkIndex

1. 적시에 방어적 복사본을 만들라
- 외부 공격으로부터 인스턴스의 내부를 보호하려면 생서자에서 받은 가변 매개변수 각각을 방어적으로 복사해야 한다.

```java
public Period(Date start, Date end) {
	this.start = new Date(start.getTime());  // 생성자를 수정해서 매개변수의 방어적 복사본을 만든다.
	this.end = new Date(end.getTime());

	if (this.start.compareTo(this.end) > 0) {
		throw new IllegalArgumentException();
	}
}
```

- 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안된다. 단 성생저와 달리 접근자 메서드에서는 방어적 복사에 clone을 사용해도 된다.
- 방어적 복사에는 성능 저하가 따르고 또 항상 쓸 수 있는 것도 아니다.  복사 비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음이 신뢰된다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때의 책임을 클라이언트에 있음을 문서에 명시하도록 하자

1. 메서드 시그니처를 신중히 설계하라
- 메서드 이름을 신중히 짓자. 표준 명명 규칙을 따른다
- 편의 메서드를 너무 많이 만들지 말자
- 매개변수 목록은 4개 이하가 좋다
    - 매개변수를 짧게 줄여주는 기술
        - 여러 메서드로 쪼갠다. List인터페이스가 그렇다
        - 잘못하면 메서드가 너무 많아질 수 있지만, 직교성을 높혀서 메서드 수를 줄여줄 수도 있다(직교성이 높다는 것은 공통점이 없는 기능들이 잘 분리되어있다는 뜻)
    - 매개변수 여러개를 묶어주는 도우미 클래스를 만드는 것으로 정적 멤버 클래스를 말한다.
    - 앞의 두개를 혼합한 것
- 매개변수 타입으로는 클래스보다는 인터페이스가 낫다.
    - 인터페이스가 있다면 직접 사용하자. 예를들어 HashMap을 넘기지 말고 Map을 사용하자. 그러면 TreeMap, ConcurrentHashMap, TreeMap등도 인수로 건낼 수 있다.
- boolean보다는 원소 2개짜리 열거 타입이 낫다.
    - 메서드 이름상 boolean을 받아야 의미가 더 명확할 때는 예외다)

52.다중정의는 신중히 사용하라

- 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택된다.
- 안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자.
    - 가변인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다.
- 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다.
    - -XLint:overloads를 지정하면 이런 종류의 다중정의를 경고해준다.
- 인수를 포워드하여 두 메서드가 동일한 일을 하도록 보장할 필요가 있다.
    - String클래스의 contentEquals메서드는 CharBuffer, String, StringBuilder, StringBuffer 등등 비슷한 부류의 타입을 가지고 있고 이들의 공통 인터페이스로 CharSequence가 등장했다.
    
    ```java
    public boolean contentEquals(StringBuffer sb) {
    	return contentEquals((CharSequence) sb);
    }
    ```
    
    - String클래스의 valueOf(char[])과 valueOf(Object)는 같은 객체를 건네더라도 전혀 다른 일을 수행한다.

1. 가변인수는 신중히 사용하라
- 가변인수는 인수 개수가 정해지지 않았을 때 유용하다.
- 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화하는 비용이 발생한다.
    - 인수 3개까지는 다중정의 메서드를 만들고 4개부터 가변 인수로 처리하는 방법으로 성능을 개선해볼 수 있다.

1. null이 아닌 빈 컬렉션이나 배열을 반환하라
- null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어나기 때문에 빈 컬렉션이나 빈 배열로 반환하도록 한다.
- Collections.emptyList, Collections.emptySet, Collections.emptyMap : 빈 컬렉션 할당이 성능을 떨어뜨릴 수 있기 때문에, 불변 컬렉션으로 반환하도록 한다
- 배열도 null을 반환하지 말고 길이가 0인 배열을 반환하라
    - 그렇다고 미리 할당하지 말자. 성능이 나빠진다.

1. 옵셔널 반환은 신중히 하라
- 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 경우보다 오류 가능성이 작다.
- 옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자
    - Optional.of(value)에 null을 넣으면 npe가 발생하니 Optional.ofNullable(value)를 사용하면 된다.
- 옵셔널은 검사 예외와 취지가 비슷하다. 즉 반환값이 없을 수도 있음을 API사용자에게 명확히 알려준다.
- 옵셔널 활용
    - 기본값을 정해둘 수 있다
    
    ```java
    String lastWordInLexicon = max(words).orElse("단어 없음...");
    ```
    
    - 원하는 예외를 던질 수 있다
    
    ```java
    Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
    ```
    
    - 항상 값이 채워져 있다고 가정한다.
    
    ```java
    Element lastNobleGas = max(Elements.NOBLE_GASES).get();
    ```
    
- 적합한 메서드를 찾지 못했다면 isPresent를 활용하자.
- 자바9에서는 Optional에 stream()메서드가 추가되었다.
- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.
    - Optional<List<T>>를 반환하기보다는 빈 List<T>를 반환하는게 좋다.
- 박싱된 기본 타입을 담는 옵셔널은 기본타입보다 무겁다. OptionalInt, OptionalLong, OptionalDouble이 있다. 박싱된 기본타입을 담은 옵셔널을 반환하는 일은 없도록 하자. 단 Boolean, Byte, Character, Short, Float은 예외일 수 있다.
- 옵셔널을 맵의 값으로 사용하면 안된다.

1. 공개된 API요소에는 항상 문서화 주석을 작성하라
- 문서화 주석 작성법(How to Write Doc Comments)가 업계 표준 API
- 메서드가 어떻게 동작하는지가 아닌, 무엇을 하는지를 기술해야한다. (how가 아닌 what을 기술해야 한다)
- 포함해야하는 내용
    - 메서드를 호출하기 위한 전제조건
    - 메서드가 성공적으로 수행된 후에 만족해야하는 사후조건
    - 부작용에 대한 내용
- 규칙
    - '@param, '@return, '@throws태그의 설명에는 마침표를 붙이지 않는다.
    - 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다.
    - 열거타입을 문서화할 때는 상수들에도 주석을 달아야 한다.
    - 애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다.
    - 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API설명에 포함해야 한다.

## :: 일반적인 프로그래밍 원칙

1. 지연변수의 범위를 최소화하라
- 메서드를 작게 유지하고 한 가지 기능에 집중한다

```java
for (int i = 0, n = expensiveComputation(); i < n; i++) {
	// 반복 변수 i,n에서, 반복여부를 결정짓는 i의 한계값을 n에 저장해서 반복 때마다 다시 계산해야하는 비용을 없앤다.
}
```

1. 라이브러리를 익히고 사용하라
- Random을 사용하지 말고 ThreadLocalRandom으로 대체한다.(성능 좋음)
- 포트 조인 풀이나 병렬 스트림에서는 SplittableRandom을 사용한다.

1. 박싱된 기본 타입보다는 기본 타입을 사용하라.
- 기본 타입은 값만 가지고 있지만, 박싱된 기본타입은 값 + 식별성(identity)를 갖고 있다.
    - new Integer(42), new Integer(42) 로 연산하면, 두개가 다른 값으로 식별된다. 즉 같은 객체를 비교하는 것이 아니면 박싱된 기본 타입에 == 연산자를 사용할 시 오류가 빌생한다.
- 박싱된 기본타입은 null을 가질 수 있다
- 기본타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.
    - 박싱과 언박싱을 반복해서 일어나게 연산되는 경우를 만들면 안된다.
- 기본타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본타입의 박싱이 자동으로 풀린다.
- 박싱된 기본 타입을 사용하는 경우
    - 컬렉션의 원소, 키, 값으로 쓴다. 컬렉션은 기본타입을 담을 수 없으므로...
        - 예) ThreadLocal<int>는 안되고 ThreadLocal<Integer>를 써야한다.
    - 리플렉션을 통해 메서드를 호출할 때도 박싱된 기본 타입을 사용한다.

1. 다른 타입이 적절하다면 문자열 사용을 피하라
- 문자열은 다른 타입, 열거타입, 혼합타입을 대신하기에 적합하지 않다.
    - 오류가능성도 꺼지고, 파싱해야해서 느리고
    - string에서 제공하는 기능에만 의존해야한다.
- 권한을 표현하기에 적합하지 않다.
    - 예를 들어 쓰레드 지역변수를 선언할 때, 클라이언트에서 호출된 문자열로 쓰레드별 지역변수를 식별하게 되면,
        - 각 클라이언트별로 고유한 키가 있어야하고 (공유되면 안되니까)
        - 결국 보안도 취약해진다.
    - 그래서 문자열 대신 Object클래스로 선언해본다.
    
    ```java
    public final class ThreadLocal<T> {
    	public ThreadLocal();
    	public void set(T value);
    	public T get();
    }
    ```
    
1. 문자열 연결은 느리니 주의하라
- 문자열 n개를 잇는 시간은 n제곱승에 비례한다. 문자열은 불변이라서 두 문자열을 연결한다치면 양쪽의 내용을 모두 복사해야 하므로 성능 저하가 발생한다.
- String대신 StringBuilder를 사용하자

1. 객체는 인터페이스를 사용해 참조하라
- 적합한 인터페이스만 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언하라
    - 객체체의 실제 클래스를 사용해야 할 상황은 오직 생성자로 생성할 뿐이다.

```java
Set<Son> sonSet = new LinkedHashSet<>();
```

- 예외.
    - String이나 Integer같은 값 클래스의 경우 인터페이스로 설계할일이 거의 없으므로 클래스로 참조해야한다.
    - 클래스 기반으로 작성된 프레임워크의 객체들의 경우 인터페이스가 없다. OutputStream등 java.io패키지의 클래스들이 여기에 해당
    - PriorityQueue클래스는 Queue인터페이스에는 없는 comparator메서드를 제공하는데 이 메소드를 꼭 사용해야하는 경우는 예외로 사용가능하다.

1. 리플렉션보다는 인터페이스를 사용하라
- 리플렉션 단점
    - 컴파일타임 타입검사가 주는 이점을 하나도 누릴 수 없다.
    - 리플렉션을 이용하면 코드가 지저분하고 장황해진다.
    - 성능이 떨어진다.

1. 네이티브 메서드는 신중히 사용하라
- 네이티브 언어는 자바보다 플랫폼을 많이 타서 이식성도 낮고 디버깅도 어려우며 성능에도 안좋다.

1. 최적화는 신중히 하라
- 최적화 규칙
    - 하지마라
    - 완전히 명백하고 최적화되지 않은 해법을 찾을 때까지는 하지마라
    - 각각의 최적화 시도 전후로 성능을 측정하라
- 성능을 제한하는 설계를 피하라
    - 완성 후 변경하기 어려운 컴포넌트끼리 또는 외부 시스템과의 통신하는 경우(API, 네트워크 프로토콜, 영구 저장용 데이터 포맷 등)
- 프로파일링 도구를 활용한다
    - [jmh](https://www.notion.so/143008071f664ff09e7f81a154237cb4)
    
1. 일반적으로 통용되는 명명 규칙을 따르라
- 자바언어명세 jls, 6.1
- 패키지명
    - 8자 이하의 짧은 단어로 한다.
        - utilities보다는 util
    - 여러 단어로 구성된 이름이면 약어로 해도 된다.
- 타입 매개변수
    - 보통 한 문자로 표현한다
        - T : 임의의 타입
        - E : 컬렉션 원소 타입
        - K, V : 맵의 키와 값
        - X : 예외
        - R : 메서드 반환 타입
        - T, U, V : 그 외에 임의 타입 시퀀스
- 문법 규칙
    - 패키지 : 따로 규칙 없다.
    - 클래스
        - 객체를 생성할 수 있는 클래스(열거타입 포함) 의 이름은 보통 단수 명사나 명사구를 사용
            
            ex) Thread, PriorityQueue, ChessPiece 등
            
        - 객체를 생성할 수 없는 클래스의 이름은
            - 복수형 명사 : Collectors, Collections 등
            - 인터페이스 이름은
                - 클래스와 똑같이 짓는다 : Collection, Comparator
                - 형용사로 짓는다 : -able, -ible
                    
                    ex) Runnable, Iterable, Accessible 등
                    
    - 메서드
        - 동사나 목적어를 포함한 동사구로 짓는다
        - boolean을 반환하는 메서드면
            - is, has + 명사/명사구/형용사로 기능하는 단어나 구로 구성한다
                
                ex) isDigit, isEmpty, isEnabled, hasSiblings
                
        - 해당 인스턴스의 속성을 반환하는 메서드
            - 명사, 명사구, get으로 시작하는 동사구로 짓는다.
        - 객체의 타입을 바꿔서 반환하는 메서드
            - to + Type형태로 한다.
                
                ex) toString, toArray등
                
        - 객체의 내용을 다른 뷰로 보여주는 메서드
            - as + Type 형태로 한다.
                
                ex) asList
                
        - 객체의 값을 기본 타입 값으로 반환하는 메서드
            - type + Value형태로 한다
                
                ex) intValue
                
        - 정적 팩터리의 경우
            
            ex) valueOf, instance, getInstance, newInstance, getType, newType
            

## :: 예외

1. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라
    1. 런타임 예외의 대부분은 전제조건을 만족하지 못했을 때 발생한다. API명세에 맞는 제약을 지키지 못했다는 뜻이다. 이 경우 복구할 수 있는 상황인지를 구분할 수 없다.  복구가 가능하다면 검사 예외를 그 외에는 런타임 예외를 사용하자
2. 필요없는 검사 예외 사용은 피하라
- API를 제대로 사용해도 발생할 수 있는 예외이거나, 프로그래머가 의미 있는 조치를 취할 수 있는 경우 외에는 비검사 예외를 사용하는 것이 좋다.

1. 표준 예외를 사용하라
- 표준 예외를 재사용하라 - 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스 적재하는 시간도 적게 걸린다.
- 대표되는 재사용 표준 예외
    - IllegalStateException
        - 인수값이 무엇이었든 어차피 실패했을 것라면 IllegaStateException
        - 그렇지 않으면 IllegalArgumentException
    - ConcurrentModificatonException : 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려고 할 때
    - UnsupportedOperationException
- 예외
    - Exception, RuntimeException, THrowable, Error는 직접 재사용하지 말자. 이 예외들은 추상 클래스라고 생각하자

1. 메서드가 던지는 모든 예외를 문서화하라
- 검사 예외는 항상 따로따로 선언하고, 각 예외가 발생하는 상황을 자바독의 '@throws태그를 사용하여 정확히 문서화 하자
    - 메서드가 Exception이나 Throwable을 던진다고 선언해서는 안된다. 메서드는 사용자에게 각 예외를 대처할 수 있는 힌트를 줘야한다. 같은 맥락에서 발생할 여지가 있는 다른 예외들까지 삼켜버릴 수 있어 API 사용성을 크게 떨어뜨린다.
- 메서드가 던질 수 있는 예외를 각각 '@throws태그로 문서화하되, 비검사 예외는 메서드 선언의 throws목록에 넣지말자. 그 이유는 검사냐 비검사냐에 따라 API 사용자가 해야 할 일이 달라지므로 두개를 확실히 구분해주는 것이 좋기 때문이다.

1. 가능한 한 실패 원자적으로 만들라
- 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 한다.
- 실패 원자적으로 만드는 방법
    - 불변 객체로 설계하는 것 : 실패하더라도 변하지 않으므로 객체가 불안정한 상태에 빠지지 않는다.
    - 작업 수행에 앞서 매개변수의 유효성을 검사하는 것
    - 객체의 임시 복사본에서 작업을 수행한 다음, 작업이 성공적으로 완료되면 원래 객체와 교체하는 것
    - 작업도중 발생하는 실패를 가로채는 복구 코드를 작성하여 작업 전 상태로 되돌리는 방법

1. 예외를 무시하지 말라.
- 예외를 무시하기로 했다면 catch 블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 바꿔놓도록 하자.

```java
try {

} catch (TimeoutException | ExecutionException ignored) {
	// 기본값을 사용한다
}
```

## :: 동시성

1. 공유중인 가변 데이터는 동기화해 사용하라
- 언어 명세상 long과 double 외의 변수를 읽고 쓰는 동작은 원자적(atomic)이다. 즉, 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 뜻이다.
- 동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.
- 여러 스레드가 가변 데이터를 공유한다면 그 데이터는 읽고 쓰는 동작은 반드시 동기화해야한다. 아니면 불변 데이터만 공유하거나 아무것도 공유하지 말자. 가변 데이터는 단일 스레드에서만 쓰도록 하자
1. 과도한 동기화는 피하라
- 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출하면 안된다
- 클라이언트가 넘겨준 함수 객체를 호출해서도 안된다
- 동기화 영역에서는 가능한 한 일을 적게 하는 것

1. 쓰레드보다는 실행자, 태스크, 스트림을 애용하라
- 실행자
    - 가벼운 서버라면 Executors.newCachedThreadPool이 일반적으로 좋다
        - CachedThreadPool은 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행된다. 그래서 무거운 어플리케이션엔 맞지 않다
    - 무거운 서버라면 Executors.newFixedThreadPool을 사용하거나 ThreadPoolExecutor를 직접 선언하도록 한다.
    - 자바7에서부터 실행자 프레임워크는 fork-join태스크를 지원한다.
- 태스크
    - 태스크는 실행자 프레임워크에서 작업단위를 나타내는 추상 개념으로 Runnable, Callable이 있다.
    - 포크 조인 태스크
        - 포크 조인풀이라는 특별한 실행자 서비스를 실행한다.
        - ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있고일을 먼저 끝넨 태스크는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수 있다.

1. wait와 notify보다는 동시성 유틸리티를 애용하라
- 동시성 유틸리티의 범주
    - 실행자 프레임워크
    - 동시성 컬렉션(concurrent collection)
        - List, Queue, Map 과 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다.
        - 높은 동시성을 수행하기 위해 각자 내부에서 동기화를 수행한다.
        - 따라서 동시성 컬렉션에서 동기화를 사용안하게 하는 것은 불가능하고 외부에서 락을 걸면 오히려 성능이 떨어진다.
        - Collections.synchronizedMap보다 ConcurrentHashMap을 사용하는 것이 훨씬 좋다. 동기화된 맵을 동시성 맵으로 교체하는 것만해도 성능이 극적으로 개선된다.
    - 동기화 장치(synchronizer)
        - 스레드가 다른 스레드를 기다릴 수 있게 해서, 서로 작업을 조율할 수 있게 해준다. (CountDownLatch, Semaphore)
- wait와 notify를 써야한다고 하면, wait는 항상 표준 관용구에 따라 while문안에서 호출하도록하자
- 일반적으로 notify보다는 notifyAll을 사용해야 한다.

1. 쓰레드 안전성 수준을 문서화하라
- 스레드 안전성이 높은 순서
    - 불변 ('@Immutable): 이 클래스의 인스턴스는 상수처럼 외부 동기화가 필요없다(String, Long, BigInteger..등)
    - 무조건적 스레드 안전 ('@ThreadSafe): 인스턴스는 수정될 수 있으나, 내부에서 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다(AtomicLong, ConcurrentHashMap)
    - 조건부 스레드 안전 : 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다.(Collections.synchronized 래퍼 메서드가 반환한 컬렉션들)
    - 스레드 안전하지 않음('@NotThreadSafe) : 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 매커니즘으로 감싸야한다 (ArrayList, HashMap 같은 기본 컬렉션)
    - 스레드 적대적 : 이 클래스는 모든 메서드 호출이 멀티 스레드에 안전하지 않다. 이 경우 deprecated API로 지정하거나 한다. (generateSerialNumber메서드가 없으면 해당)
- 반환 타입만으로 명확히 할 수 없는 정적 팩터리라면 객체의 스레드 안전성을 문서화해야한다
- 열거 타입은 굳이 불변으로 안써도 된다

1. 지연 초기화는 신중히 사용하라
- 지연 초기화란?
    - 필드이 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법
- 지연 초기화는 양날의 검이다. 클래스 또는 인스턴스 생성 시의 초기화 비용은 줄지만 지연 초기화하는 필드에 접근하는 비용은 커진다.
- 멀티 스레드 환경에서는 지연 초기화해야하는 필드를 둘 이상의 스레드가 공유한다면 반드시 동기화해야 한다.
- 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.
- 지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용하자

```java
private FieldType field;

private synchronized FieldType getField() {
	if (field == null) 
			rield = computeFieldValue();
	return field;
}
```

- 성능 때문에 정적 필드를 지연 초기화해야한다면 지연 초기화 홀더 클래스 관용구를 사용하자

```java
private static class FieldHolder {
	static final FieldType field = computeFieldValue();
}

private static FieldType getField() {
	return FieldHolder.field;
}

// getField가 호출되는 순간 Fieldholder.field가 처음 읽히면서, FieldHolder클래스 초기화가 이뤄진다.
// 이 관용구는 getField메서드가 필드에 접근하면서 동기화를 전혀하지 않으니 성능이 느려질게 없다.
```

- 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하라.
    - 한번은 동기화 없이 검사하고, 한번은 동기화해서 검사한다. 두번 째 검사에서도 필드가 초기화되지 않았을 때만 초기화하도록 한다.
    - 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatile로 선언해야한다.

```java
private volatile FieldType field;

private FieldType getField() {
	FieldType result = field;

	if (result != null) {    // 초기화된 상황에서는 필드를 한번만 읽도록 보장. 첫번째 검사
		return result;
	}

	synchronized(this) {
		if (field == null) {    // 두 번째 검사
			field = computeFieldValue();
		}
		return field;
	}
}
```

- 대부분의 필드는 지연 초기화가 필요없다. 꼭 지연 초기화를 써야한다면 인스턴스 필드에는 이중검사 관용구를, 정적 필드에는 지연 초기화 홀더 클래스 관용구를 사용하자.

1. 프로그램의 동작을 스레드 스케쥴러에 기대지 말라
- 여러 스레드가 실행중이면 운영체제의 스레드 스케쥴러가 어떤 스레드를 얼마나 오래 실행할지 결정한다. 정확성이나 성능이 스레드 스케쥴러에 따라 달라지는 프로그램이라면 다른 플랫폼에 이식하기 어렵다
- 실행 가능한 스레드 수를 프로세서 수보다 지나치게 많아지지 않게 하는 것이 중요하다.
- 스레드가 작업이 완료되면, 다음 작업이 있을 때까지 대기하게 하는 것이 좋다. 즉 스레드 풀을 적절하게 설정하고 작업은 짧게 유지한다.
- 스레드는 busy waiting 상태가 되면 안된다. 이 상태는 스레드 스케쥴러에 취약하고 프로세스에 큰 부담을 준다.
- 스레드 우선순위나 Thread.yield로 해결하려고 하면 안된다. 이식성도 떨어지고 합리적인 조치가 아니다.

## ::직렬화

1. 자바 직렬화의 대안을 찾아라
- 직렬화 위험을 회피하는 가장 좋은 방법은 아무것도 역직렬화하지 않는 것이다.
    - 크로스-플랫폼 구조화된 데이터 표현
        - json : 텍스트 기반 표현에는 효과적
        - 프로토콜 버퍼
- 자바 직렬화를 피할 수 없다면, 신뢰할 수 없는 데이터는 절대 역직렬화하지 않는 것이다.
    - 객체 역직렬화 필터링(java.io.ObjectInputFilter)을 사용하자
1. Serializable을 구현할지는 신중히 결정하라
- Serializable을 구현하면 릴리스한 뒤에는 수정하기 어렵다. 직렬화 형태도 하나의 공개 API가 된다.
- 직렬화가 클래스 개선을 방해하는 예
    - 스트림 고유 식별자, 즉 직렬 버전 UID가 없으면 시스템이 런타임에 암호해시 함수를 적용해 자동으로 클래스 안에 생성해 넣는다. 그런데 자동으로 생성되면 나중에 클래스가 수정되면 uid값도 변경되기 때문에 호환성이 꺠져버려 런타임에 InvalidClassException이 발생한다.
    - 버그와 보안 구멍이 생길 위험이 높아진다.
        - 직렬화는 언어의 기본 매커니즘을 우회하는 객체 생성 기법인 것이다. 역직렬화는 일반 생성자의 문제가 그대로 적용되는 '숨은 생성자'다. 기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다는 뜻이다.
    - 해당 클래스의 신 버전을 릴리스할 때 테스트할 것이 늘어난다는 것이다.
    - 상속용으로 설계된 클래스는 대부분 Serializable을 구현하면 안되며, 인터페이스도 대부분 Serializable을 확장해서는 안된다.
    - 내부클래스는 직렬화를 구현하지 말아야 한다.

1. 커스텀 직렬화 형태를 고려해보라
- 자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때만 사용하고, 그렇지 않으면 객체를 적절히 설명하는 커스텀 직렬화 형태를 고안하라
- 구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 uid를 절대 수정하지말자.

1. readObject 메서드는 방어적으로 작성하라
- readObject메서드를 작성할 때는 항상 public생성자를 작성하는 자세로 해야한다.
- readObject는 어떤 바이트 스트림이 넘어오더라도 유효한 인스턴스를 만들어내야한다. 바이트 스트림이 진짜 직렬화된 인스턴스라고 가정해서는 안된다.
1. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라.
- 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 한 열거 타입을 사용하자

```java
public enum Elvis {
	INSTANCE;

	private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel"};

	public void printFavorites() {
		System.out.println(Arrays.toString(favoriteSongs));
	}
}
```

- implement Serializable을 추가하면 싱글턴이 아니게 되는데, readResolve기능을 이용하면 readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다. 그리고 그 클래스에서 모든 참조 타입 인스턴스 필드를 transient로 선언해야한다.

1. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라
- 직렬화를 구현하게 되면 버그와 보안 문제가 일어나게 되는데 그 위험을 줄여줄 기법으로 직렬화 프록시 패턴(serialization proxy pattern)이 있다.

```java
private Object writeReplace() {  // 직렬화 프록시 패턴용 writeReplace 메서드
	return new SerializationProxy(this);
}
```

- 직렬화 프록시 패턴 단점
    - 클라이언트가 확장할 수 있는 클래스에는 적용할 수 없다
    - 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다.
    - 방어적 복사보다 성능이 안좋다.