---
layout: post
title: about HHH000038, HHH000039 warning
date: 2024-06-27 10:19:00
last_modified_at : 2024-06-27 10:19:00
parent: DB
has_children: false
nav_order: 1
nav_exclude: true
---

# problem

- sprinig boot start 하면 warning  레벨로 아래 로그가 로깅된다.

```prolog
WARN [main] org.hibernate.mapping.RootClass        
:287 ositeIdentifier HHH000038: Composite-id class does not override equals(): XxxJpoId
WARN [main] org.hibernate.mapping.RootClass         
:290 ositeIdentifier HHH000039: Composite-id class does not override hashCode(): XxxJpoId
```

# cause

- HHH000038,  HHH000039 Composite-id class does not override equals(), hashCode() 메시지는 Hibernate에서 발생시키는 에러다.
- 에러가 발생한 클래스는 @Embeddable을 설정한 클래스로 JPA 비지니스에서 사용할 오브젝트 클래스인데, 키에 해당하는 필드가 여러개로 되어있음에도 equals() , hashCode() 메서드를 재정의하지 않았다.
    - 복합키로 구성했을 경우 재정의하지 않으면 다른 문제가 발생할 수 있다.
- hibernate는 엔티티 인스턴스의 동일성(identity)을 체크할 때 equals()와 hashCode() 메서드를 사용하기 때문이다.

# solved

- 방법 1) 해당 엔티티 클래스에 equals()와 hashCode() 메서드를 정의
- 방법 2) 해당 엔티티 클래스에 @EqualsAndHashCode 추가
    
    ```java
    @Embeddable
    @EqualsAndHashCode
    public class XxxJpoId implements Serializable {
        private String xxxId;
    ```