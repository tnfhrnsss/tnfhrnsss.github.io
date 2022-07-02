---
layout: post
title: 리팩토링 3장
date: 2015-02-24 00:00:00
last_modified_at : 2015-02-24 00:00:00
parent: Refactoring
grand_parent: Java
has_children: false
nav_order: 2
nav_exclude: true
tags: [refactoring]
---

3. 코드의 구린내 ( 부록D 구린내와 탈취기법 491p 참조 )
리팩토링을 해야하는 시점과 기준
사람의 직관. (충분한 연습을 통해 감을 잡는 방법)


3.1. 중복 코드(uplicated Code)
구린내의 제왕.

- 한곳 이상에서 중복된 코드가 나타날때 Extract Method(142p)
- 한 클래스의 두 하위클래스에서 같은 코드가 나타나는경우 Extract Method(142p)후 Pull Up Method(381p)
- 코드가 똑같지 않고 비슷하다면 Extract Method(142p)로 같은 부분과 다른부분을 분리후 경우에 따라 Form Template method(407p)
- 메서드들이 같은 작업을 하지만 다른 알고리즘을 사용한다면 두 알고리즘중 더 명확한 것을 선택해 Substitute Algorithm(174p)
- 서로 관계가 없는 두 클래스에서 중복된 코드가 있는경우
  한 클래스 안의 중복코드를 Extract Class(187p), Extract Method(142p)후 생성된 메서드를 다른 클래스에서 호출



3.2 장황한 메소드(Long Method)
메서드를 과감하게 쪼게고, 메서드의 기능을 한눈에 알 수 있는 메소드명 사용.

- 대부분의 경우 메서드의 길이를 줄이기 위해 하는것은 Extract Method(142p) 이다.
- 메서드에 매개변수와 임시변수가 많다면, 메서드를 추출하기 어렵다.
  Replace Temp with Query(153p 임시변수를 메소드 호출로 변환)기법 적용으로 임시변수 제거
- 긴 매개변수 리스트는 Introduce Parameter Object(351p : 매개변수 세트를 객체로 전달),
  Preserve Whole Object(342p: 객체를 통째로 전달)로 갈결하게함.
- 그래도 매개변수와 임시변수가 많다면 Replace Method with Emthod Object(170p 메서드를 메서드 객체로 전환)

조건문과 루프 또한 메서드 추출이 필요
- 조건문을 추출하려면 Decompose Conditional(286p 조건문 쪼개기) 사용



3.3 거대한 클래스(Large Class)
메서드를 과감하게 쪼게고, 메서드의 기능을 한눈에 알 수 있는 메소드명 사용.

- 대부분의 경우 메서드의 길이를 줄이기 위해 하는것은 Extract Method(142p) 이다.
- 메서드에 매개변수와 임시변수가 많다면, 메서드를 추출하기 어렵다.
  Replace Temp with Query(153p 임시변수를 메소드 호출로 변환)기법 적용으로 임시변수 제거
- 긴 매개변수 리스트는 Introduce Parameter Object(351p : 매개변수 세트를 객체로 전달),
  Preserve Whole Object(342p: 객체를 통째로 전달)로 갈결하게함.
- 그래도 매개변수와 임시변수가 많다면 Replace Method with Emthod Object(170p 메서드를 메서드 객체로 전환)

조건문과 루프 또한 메서드 추출이 필요
- 조건문을 추출하려면 Decompose Conditional(286p 조건문 쪼개기) 사용

3.3 방대한 클래스(Large Class)
- 지나치게 많은 인스턴스 변수가 나타나는 클래스는 Extract Class(187p 클래스 추출)후 많은 인스턴스 변수를 하나로 묶을수 있다.
  클래스내에서 서로에게 의미가 있는 변수를 골라서 묶는다. 클래스안에서 변수들이 공통적인 접두사,접미사어가 같으면 클래스를 따로 뽑아낸다.
  하위클래스로 추출하는 것이 더 적합하면 Extract Subclass(390p 하위클래스 추출)이 더 간단.

- 코드량이 방대한 클래스는 Extract Class(187p), Extract Subclass(390p)
  클라이언트가 클래스를 어떻게 쓰게 할것인지를 결정하고 각각의 사용방법에 대해 Extract Interface(403p 인터페이스 추출)를 사용
  --> 클래스를 어떻게 분해할지에 대한 아이디어를 줄것이다.

- 방대한 클래스가 GUI클래스라면 Duplicate Observed Data(231p 관측데이터 복제)




3.4 긴 파라미터 리스트(Long Parameter List)
이미 알고있는 객체에 요청하여 매개변수의 데이타를 얻을 수 있으면 Replace Parameter with Method(347p 매개변수 세트를 매서드로 전화)를 사용.
한 객체로부터 가져온 데이터세트를 객체 자체로 바꾸기위해 Preserve Whole Object(342p 객체를 통째로 전달) 사용
여러 데이터 항목에 논리적 객체가 없다면 Introduce Parameter Object(351p 매개변수 세트를 객체로 전환)



3.5 수정의 산발(Divergent Change)
한클래스가 다른 원인으로 인해 다양한 방법으로 자주 수정되는 경우에 발생 --> 두개의 클래스로 나누어 하나의 클래스만 수정
Extract Class(187p)를 사용하여 하나의 클래스로 묶음



3.6 기능의 산재(Shotgun Surgery)
수정의 산발과 비슷하지만 정반대. 변경을 할때마다 많은 클래스를 수정하는경우

- move Method(178p 메서드 이동), Move Filed(183p 필드 이동) 을 사용하여 수정할 부분을 모두 하나의 클래스로 안에 넣음.
- 대개는 Inline Class(192p 클래스 내용 직접 삽입)를 사용하여 분산된 기능을 하나로 모음.




3.7 잘못된 소속 (Feature Envy)
어떤 값을 계산하기 위해 다른 객체에 잇는 여러개의 get 메소드를 호출하는 경우


- 그 메소드는 분명히 다른곳에 있고 싶은것이고 따라서 Move Method(170 메소드 이동)을 사용한다.

- 때로는 메소드의 특정부분만 이런 욕심으로 고통받는데 이럴때는 욕심이 많은 부분에 대해서 Extract Method(136 메소드로 추출)을 사용한담음 Move Method 를 사용



3.8 데이타 덩어리(Data Clump)
함께 몰려다니는 데이터의 무리는 그들 자신의 객체로 만들어져야한다.


- 이 덩어리를 객체로 바꾸기 위해 이 필드들에 대해 Extract CLass(179)를 사용한다.
- 그런다음 관심을 메소드 시그너처로 돌려 Introduce parameter Object(339: 파라미터를 객체로 전환)나 Preserve Whole Obejct(331 파라미터값을 구하는 객체자체를 마라미터로 던진다)을 사용하여 파라미터 리스트를 단순하게 한다.


3.9 기본 타입에 대한 강박관념(primitive Obsession)
기본타입과 레코드타입을 객체로 바꿔라.


- 각각의 데이타 값에 대해 Replace Data Value with Object(209 : 추가적인 데이타나 동작을 필요로 하는 데이타아이템이 있을경우 데이타를 객체로 바꿔라)를 사용
- 데이터 값이 타입 코드이고, 값이 동작에 영향을 미치지 않는다면 Replace Type Code with Class(255 : 클래스의 동작에 영향을 미치지 않는 숫자로 된 타입코드가 있으면 숫자- 를 클래스로 바꾸어라)를 사용
- 만약 타입타입코드에 의존하는 조건문이 있는 경우에는 Replace Type Code With Subclass , Replace Type Code With State/Strategy(265)를 이용
- 항상 몰려다녀야 할 필드 그룹이 있따면 Extract Class(179)
- 파라미터 리스트에서 이런 기본타입을 보면 Introduce Parameter Object(339)를 사용하라.
- 배열을 쪼개서 쓰고있는 자신을 발견하거든 Replace Array with Object(220)을 사용하라.


3.10 Switch문(Switch Statements)
siwtch 문의 본질적인 특징은 중복된다는점
항상 다형성을 생각.


- Extract Method(136)을 사용하여 switch문을 뽑아내고, Move Method(170)를 사용하여 다형성이 필요한 클래스로 옮긴다
- 이시점에서 Replace Type Code with Subclasses(261)를 사용할것인지, Replace Type Code with State/Strategy(265)를 사용할것인지 결정해야한다.
- 상속구조를 결정했으면 Replace Conditional with Polymorphism(293)을 사용할수 있다.
- 만약 하나의 메소드에만 영향을 미치는 몇개의 경우가 있따면 굳이 바꿀 필요가 없다. Replace Parameter with Explict Methods(327)
- 조건중 null 이 있는경우 Introduce Null Object(298)


3.11 평행 상속 구조(Parallel Inheritance Hierarchies)
산탄총 수술의 특별한 경우
한 클래스의 서브클래스를 만들면 다른곳에도 모두 서브클래스를 만들어주어야하는경우
중복을 제거하는 일반적인 전략은 한쪽 상속구조의 인스턴스가 다른쪽 구조의 인스턴스를 참조하도록 만드는것


- Move Method(170), Move Filed(175)를 사용


3.12 게으른 클래스(Lazy Class)
충분한 일을 하지 않는 클래스는 삭제되어야한다.


- 별로 하는일도 없는 클래스의 서브클래스는 Collapse Hierarchy(392), 거의 필요없는 클래스는 Inline Class(184)


3.13 막연한 범용코드(Speculative Generality)
- 별다른 기능이 없는 클래스나 모듈은 Collapse Hierarchy(406p 계층 병합)
- 불필요한 위임은 Inline Class(192p)
- 메소드에 사용되지 않는 매개변수가 있다면 Remove Parameter(329p)
- 메소드 이름이 이상하면 Rename Method(325p)


3.14 임시필드(Temporary Field)
사용되지 않는 변수가 왜 있는지를 이해하려는것은 짜증나는일
임시필드는 복잡한 알고리즘이 여러변수를 필요로 할때 흔히 나타남.


- Extract Class(187p), Intoduce Null Object(310p)


3.15 메시지 체인 (Message Chains)
클라이언트가 객체의 클래스 구조와 결합되어있을때, 객체가 객체에 물어보고.. 이 객체는 다른객체에.. 또 이객체는 다른 객체에..


- Hide Delegate(195p)을 사용.
- 객체가 사욧ㅇ되는 코드 부분을 Extract Method(142p)후 Move Method(178p)로 체인 아래로 밀어낼 수 있는 지 여부 검사 후 실시

3.16 과잉 중개 메서드(Middel Man)
메소드의 태반이 다른 클래스로 위임을 하고 있다면

- Remove Middle Man(198p 클라이언트가 대리 객체를 직접 사용)을 실시.
- 일부 메소드에 별 기능이 없다면 Inline Method(150p)를 실시.
- 추가동작 필요시 Replace Delegation with Inheritance(404) 실시. 해당 메소드 내용을 호출 객체에 직접 삽입
- 부수적인 기능이 있다면 Replace Inheritance with Deligation
(416p 상위클래스에 필드를 작성하고, 모든 메서드가 그 상위클래스에 위임하게 수정한 후 하위클래스 제거)을 적용


3.17 지나친 관여(Inappropriate Intimacy)
클래스끼리의 관계가 지나치게 밀접하게 되어 서로의 private 부분을 알아내느라 과도한 시간을 낭비할 때.


- Move Method(178p),Move Field(183p)를 실시, 각 클래스를 분리해서 지나친 관여를 줄인다.
- 클래스의 양방향 연결을 단방양으로 전환Change Bidirectional Association to Unidirectional(244p)이 적용가능한지 살피고,
- 공통으로 필요한 부분이 있다면 Extract Class(187p)를 사용하여. 공통된 부분을 별도의 클래스로 생성
- 또는 Hide Delegate(195p)을 사용하여 다른 클래스가 중개 메서드 역할을 하게 만듬.


3.18 인터페이스가 다른 대용 클래스 (Alternatice Classes with Different Interface)
- 기능은 같은데 다른 시그너쳐를 가지고 있는경우는 Rename Method(325p)을 실시
- 포로토콜이 같아질때까지 Move Method(178p)를 실시하여 기능을 해당클래스에 이동
- 너무 많은 코드를 옮겨야 할 경우 Extract Superclass(397p)를 사용.


3.19 미흡한 라이브러리 클래스(Incomplete Library Class)
라이브러리 클래스를 수정하는 것은 보통은 불가능하다.


- 라이브러리 클래스가 가지고 있었으면 하는 메소드가 몇개 있다면, Introduce Foreign Method(201p)를 사용하라.
- 부가 기능이 많을 경우 Introduce Local Extension(203p)이 필요하다.


3.20 데이터 클래스(Data Class)
데이타 보관만 담당. 거의 대부분 다른 클래스에 의해 조작된다.

- public 필드는 Encapsulate Field(250p) 적용
- Collection 필드는 Encapsulate Collection(252p) 적용
- 변경되면 안되는 필드에 대해서는 Remove setting Method(357p) 적용
- get/set 메소드가 다른 클래스에서 사용될때 Move Method(178p), Extract Method(142p)사용하여 데이터 클래스로 옮긴후
  get/set메소드에 Hide Method(357p) 적용



3.21 방치된 상속물(Refused Bequest)
하위클래스가 기능은 재사용하지만 상위클래스의 인터페이스를 지원하지 않을 때 구린내가 심함.

- 계층 구조를 손보는 것보다는 Replace Inheritance with Deligation(416p)을 적용


3.22 주석 (Comments)
주석을 써야 할 것 같은 생각이 들면, 먼저 코드를 리팩토링하여 주석이 불필요하도록 하라.

- 코드 구간의 기능을 설명할 수적이 필요하ㄹ 때는 Extract method(142p)를 실시.
- 그래도 주석이 필요하다면 Rename Method(325p) 사용
- 시스템의 필수적인 상태에 대해 약간의 규칙을 설명할 필요가 있다면 Introduce Assertion(319p)을 사용.