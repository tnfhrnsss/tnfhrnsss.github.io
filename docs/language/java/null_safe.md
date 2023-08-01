---
layout: post
title: Safely handling 'null' in Java
description: Safely handling 'null' in Java
date: 2023-08-01 09:16:00
last_modified_at : 2023-08-01 09:16:00
parent: Java
grand_parent: Language
has_children: false
nav_exclude: true
---

[https://youtu.be/vX3yY_36Sk4](https://youtu.be/vX3yY_36Sk4)

## 1. null을 최대한 쓰지 말라

- null로 지나치게 유연한 메서드를 만들지 말고 명시적인 메서드를 만들어라
- null을 반환하지 말라
    - 반환 값이 꼭 있어야 한다면 null을 반환하지 말고 예외를 던져라
    - 빈 반환 값은 빈 컬렉션이나 Null 객체를 활용하라
    - 반환 값이 없을 수도 있다면 Optional을 반환하라.
- 선택적 매개변수는 null 대신 다형성을 사용해서 표현하라
- Null 객체란?
    - type safe하면서 의미를 표현할 수 있는 동일한 타입의 특수 객체를 만들어서 반환
    - 다형성 활용
    - 아무일도 하지 않는 객체 → 더미 객체
    - 리스코프 치환원칙 주의

## 2. 계약에 의한 설계를 하라 (Design by Contract)

- solid의 개방 폐쇄 법칙의 상위개념으로, API 규약을 소비자와 제공자 사이에 지켜야 할 엄격한 계약으로 여기는 설계방법을 말한다.
- 즉, null이 들어왔을때 null이 들어왔는지 확인하는 작업을 말한다.
- 예를 들면
    - Interface + Java Doc
    - 사전조건 = 보호절 (guard clause)
        - assertion
        - Objects의 메서드
        - IllegalArgumentException, NullPointException

## 3. null의 범위를 지역으로 제한하라.

- 상태 프로세스를 메서드 안으로 숨기고 전파시키지 말아야 side effect가 발생하지 않는다.
- 기본 문제 해결 원칙은 큰문제를 제어가능한 작은 문제로 나누고 다시 통합하는 것
- 상태 프로세스와 마찬가지로 null도 메서드 안으로 제한시키면 큰 문제가 발생하지 않는다
- 클래스와 메서드를 작게 만들어라
- 설계가 잘된 코드에서는 널의 위험도 적어진다.

## 4. 초기화를 명확히 한다.

- 초기화 시점과 실행 시점이 겹치면 안된다.
- 실행 시점엔 초기화되지 않은 필드가 없어야 한다.
- 실행 시점에 null인 필드는 초기화되지 않았다는 의미가 아닌, 값이 없어야 한다는 의미여야한다
- 객체 필드의 생명 주기는 모두 객체의 생명주기와 같아야 한다.
- 지연 초기화(lazy initialization)필드의 경우 팩토리 메서드로 null처리를 캡슐화 하라