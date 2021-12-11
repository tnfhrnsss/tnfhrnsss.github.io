---
layout: default
title: Type safe heterogeneous container pattern
parent: Patterns
has_children: false
nav_order: 11
nav_exclude: true
---

# 타입 안전 이종 컨테이너 패턴

type safe heterogeneous container pattern

```java
public class Favorites {
	public <T> void putFavorite(Class<T> type, T instance);
	public <T> T getFavorite(Class<T> type);
}

public static void main(String[] args) {
	Favorites f = new Favorites();

	f.putFavorite(String.class, "Java");
	f.putFavorite(Integer.class, 0xcafebabe);

	String favoritesString = f.getFavorite(String.class);
}
```

- Favorites인스턴스는 String이 요청되면 Integer를 반환하는 일이 없기 때문에 타입안전하다고 볼 수 있다.
- 일반 적인 맵과 달리 여러가지 타입의 원소를 담을 수 있어서, 타입안전이종 컨테이너라고 한다.
- ex) DatabaseRow타입의 Column<T>

```java
public class Favorites {
	private Map<Class<?>, Object> favorites = new HashMap<>();

	public <T> void putFavorites(Class<T> type, T instance) {
		favorites.put(Objects.requireNonNull(type), type.cass(instance));
	}
}
```

- 타입을 한정시키고 싶다면?
    - ex) annotation api
    
    ```java
    public <T extends Annotation>
    		T getAnnotation(Class<T> annotationType);
    
    static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
    	Class<?> annotationType = null; // 비한정적 타입 토큰
    	try {
    		annotationType = Class.forName(annotationTypeName);
    	} catch (Exception ex) {
    		throw new IllegalArgumentException(ex);
    	}
    	return element.getAnnotation(annotationType.asSubclass(Annotation.class));
    }
    
    ```
    
    - annotationType인수는 한정적 타입 토큰