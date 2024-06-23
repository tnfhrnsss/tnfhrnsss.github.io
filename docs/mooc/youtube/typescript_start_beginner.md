---
layout: post
title: 타입 스크립트 학습 노트 (비기너) with 코딩앙마
date: 2024-06-23 16:55:00
last_modified_at : 2024-06-23 16:55:00
parent: Youtube
grand_parent: Mooc
nav_exclude: true
---


# 강의 정보

- 유튜브 : [코딩앙마의 타입스크립트](https://youtube.com/playlist?list=PLZKTXPmaJk8KhKQ_BILr1JKCJbR0EGlx0&si=HbOAizVkTMznoQyg){:target="_blank"}
- tool : [https://www.typescriptlang.org/play](https://www.typescriptlang.org/play){:target="_blank"}

# 1. 타입스크립트를 사용하는 이유

- 브라우저에서는 ts → js로 변환해야 해석될 수 있다.
- 자바스크립트는 동적언어로 런타임 시점에 타입이 결정되고 오류가 발견된다.

# 기본타입

- string, array, tuple,
- void
- never
    - 에러로 반환하거나
    - 영원히 끝나지 않을때
- enum
    - enum에 값을 할당하지 않으면, 1씩 증가하면서 할당한다.
        
        ```jsx
        enum Os {
            Window = 3,
            Ios = 10,
            Android
        }
        ```
        
    - 문자열 할당도 가능하다.
        
        ```jsx
        enum Os {
            Window = 'win',
            Ios = 'ios',
            Android = 'and'
        }
        
        let myOs:OS;
        
        myOs = Os.Window
        ```
        
- null
    - let a:null = null;
- undefined
    - let a:undefined = undefined;

# 인터페이스

```java
type Score = 'A' | 'B' |'C' | 'F';

interface User {
    name: string;
    age: number;
    gender? :string;
    readonly birthYear : number;
    [key:number] : string;
    [grade:number] : Score;
}

let user: User = {
    name: 'xx',
    age: 30
}

user.age = 10;
console.log(user.name)
```

- optional 프로퍼티는 ? 물음표를 붙인다
- readonly 읽기전용 프로퍼티는 ‘readonly’를 붙인다. 수정불가
- [key:number] : string; 는 number형을 key로 하고 value는 string인 배열을 여러개 저장할 수 있다는 의미
- [grade:number] : Score; 는 Score의 type으로 정한 값만 선언할 수 있다.
- 함수 정의할 수 있다.
    
    ```java
    interface Add {
        (num1: number, num2:number): number;
    }
    
    const add : Add = function(x, y) {
        return x + y;
    }
    
    add(10, 20);
    ```
    
- interface로 implements를 통해 class도 정의할 수 있다.
    
    ```java
    interface Car {
        color: string;
        start() : void;
    }
    
    class Bmw implements Car {
        color = "red";
    
        constructor(c:string) {
            this.color = c;
        }
    
        start() {
                
        }
    }
    
    const b = new Bmw('green');
    
    b.start();
    ```
    
- interface는 extends를 통해 확장도 가능하다.
    
    ```java
    interface Car {
        color: string;
        start() : void;
    }
    
    interface Benz extends Car {
        door: number;  // 추가 정의
        stop(): void;
    }
    ```
    
    - 여러개를 확장할 수 있다.
        
        ```java
        interface Car {
            color: string;
            start() : void;
        }
        
        interface Toy {
            name: string;
        }
        
        interface ToyCar extends Car, Toy {
            price: number;
        }
        ```
        

# 함수

- 선택적 매개변수
    - 매개변수도 optional하게 정의할 수 있다.
    
    ```java
    function hello(name?: string) {
        return `Hello, ${name || "world"}`;
    }
    
    const result = hello();
    ```
    
    - 하지만 매개변수가 여러개 일때, 선택적 매개변수는 필수 매개변수 앞에 올수 없다.
        - 다만 앞에 두고 쓸때는 undefined를 디폴트로 둘 수 있다.
        
        ```java
        function hello(age:number | undefined, name: string): string {
            if (age !== undefined) {
                return `Hello, ${name}. You ar ${age}.`;
            } else {
                return `Hello, ${name}`;
            }
        }
        
        console.log(hello(undefined, "Sam));
        ```
        
- 디폴트 매개변수를 선언할 수 있다.
    
    ```java
    function hello(name = "world") {
        return `Hello, ${name || "world"}`;
    }
    ```
    

# this

```java
interface User {
    name: string;
}

const Sam: User = {name: 'Sam'}

function showName() {
    console.log(this.name);
}

const a = showName.bind(Sam);
a();
```

- 위의 코드는  bind를 통해서 this를 Sam객체로 대체하고 있다.
- 하지만 저 상태는 this.name에서 this를 참조할 수 없다고 나온다.
    
    ```java
    function showName(this:User) {
        console.log(this.name);
    }
    ```
    
    - 매개변수로 User객체를 던지면 해결된다.
- this 외에 매개변수가 있다면
    
    ```java
    function showName(this:User, age:number, gender: 'm' | 'f') {
        console.log(this.name, age, gender);
    }
    
    const a = showName.bind(Sam);
    a(30, 'm');
    ```
    

# Overload

```java
interface User {
    name: string;
    age: number;
}

function join(name: string, age: string): string;
function join(name: string, age: number): User;
function join(name: string, age: number | string): User | string {
    if(typeof age === "number") {
        return {
            name,
            age
        };
    } else {
        return "나이는 숫자로 입력해주세요"
    }
}

const sam: User = join("Sam",30);
const jane: string = join("Jane", "30");
```

- 동일한 함수이지만 매개변수 타입이나 개수에 따라 다르게 동작한다면 오버로드를 정의해서 사용할 수 있다.

# 리터럴, 유니온/교차 타입

- literal types : const(상수), let
- Union Types (Or 의미)
    - number 유니온 타입
        
        ```java
        interface HighSchoolStudent {
            name: name | string;
            grade: 1 | 2 | 3;
        }
        ```
        
    - 식별가능한 유니온 타입
        
        ```java
        interface Car {
            name: "car",
            color: string;
            start(): void;
        }
        
        interface Mobile {
            name: "mobile";
            color: string;
            call(): void;
        }
        
        function getGift(gift: Car | Mobile) {
            console.log(gift.color);
            if (gift.name === "car") {
                gift.start();  // 마우스 오버하면 Car 객체라는 것을 알 수 있다.
            } else {
                gift.call();
            }
        }
        ```
        
- Intersection Types (And의미)
    - 여러 타입을 하나로 합쳐서 사용할때 선언한다.
    - 대신 모든 타입의 속성을 모두 정의해야한다.
        
        ```java
        interface Car {
            name: string,
            start(): void;
        }
        
        interface Toy {
            name: string;
            color: string;
            price: number;
        }
        
        const toyCar: Toy & Car = {
            name: "타요",
            start() {
        
            },
            color: "blue",
            price: 1000,
        }
        ```
        

# 클래스 Class

- 생성 자 변수를 선언해야한다.
    - public이나 readonly이면 선언하지 않아도 된다.
- 접근제한자 - public, private, protected
    - es6와 달리 타입스크립트는 접근제한자를 지원한다.
- static 변수는 this가 아닌 클래스명으로 호출한다.
    
    ```java
    class Car {
        static wheel =4;
        constructor(public color: string) {
            this.color = color;
        }
    
        start() {
            console.log("start");
        }
    }
    
    console.log(this.wheel); // -> this.wheel이 아닌 Car.wheel로 호출해야한다.
    ```
    
- 추상 클래스
    - new를 통해서 생성할 수 없고, 상속을 통해서 구현해야한다
    - 추상클래스의 추상메서드를 구현하지 않으면 오류난다
    
    ```java
    abstract class Car {
        color: string;
        constructor(color: string) {
            this.color = color;
        }
    
        start() {
            console.log("start");
        }
        abstract doSomething(): void;
    }
    
    class Bmw extends Car {
        constructor(color: string) {
            super(color);
        }
        doSomething() {
            alert(3)
        }
    }
    const bmw = new Bmw("red");
    ```
    

# Generics

- 제너릭을 사용하면, 클래스, 함수, 인터페이스를 다양한 타입으로 재사용할 수 있다.
- 선언할때 타입파라미터를 주고, 사용할때 타입이 결정된다.

```java
function getSize(arr: number[] | string[]): number {
    return arr.length;
}

const arr1 = [1, 2, 3];
getSize(arr1);

const arr2 = ["1", "2", "3"];
getSize(arr2);

const arr3 = [true, boolean];
getSize(arr2);
```

- 이렇게 사용하면 계속 유니온으로 타입을 추가할 수 밖에 없다
    
    ```java
    function getSize1<T>(arr: T[]): number {
        return arr.length;
    }
    
    const arr11 = [1, 2, 3];
    getSize1(arr11);
    
    const arr22 = ["1", "2", "3"];
    getSize1(arr22);
    
    const arr33 = [true, boolean];
    getSize1(arr33);
    ```
    
    - 타입파라미터 <T>를 선언하면 여러 타입을 사용할 수 있다.

- 인터페이스에 any타입이 추가될 경우, any 대신 타입파라미터를 활용할 수 있다.
    
    ```java
    interface Mobile<T> {
        name: string;
        price: number;
        option: T;
    }
    
    const m1: Mobile<Object> = {
        name: "s21",
        price: 1000,
        option: {
            color: "red",
            coupon: false,
        }
    }
    
    const m2: Mobile<{ color: string; coupon: boolean}> = {
        name: "s21",
        price: 1000,
        option: {
            color: "red",
            coupon: false,
        }
    }
    
    const m3: Mobile<String> = {
        name: "s20",
        price: 900,
        option: "good"
    }
    ```
    
- 여러 인터페이스에 적용할 경우
    
    ```java
    interface User {
        name: string;
        age: number;
    }
    
    interface Car {
        name: string;
        color: string;
    }
    
    interface Book {
        price: number;
    }
    
    const user: User = {name: "a", age:10};
    const car: Car = {name: "bmw", color: "red"};
    const book: Book = {price: 3000};
    
    function showName<T extends { name: string}>(data: T): string {
        return data.name;
    }
    
    showName(user);
    showName(car);
    showName(book); // book에는 name이 없으므로
    
    ```
    

# Utility Types

## 1. keyof

```java
interface User {
    id: number;
    name: string;
    age: number;
    gender: "m" | "f";

}

type UserKey = keyof User;
const uk:UserKey = "name"
```

- UserKey는 `‘id’ | ‘name’ | ‘age’ | ‘gender’` 와 같다
- uk에는  `‘id’ | ‘name’ | ‘age’ | ‘gender’` 외에 선언할 수 없다.

## 2. Partial<T>

- interface속성을 다 정의하지 않아도 되게 해준다. optional처리와 같다
    
    ```java
    interface User {
        id: number;
        name: string;
        age: number;
        gender: "m" | "f";
    
    }
    
    let admin: Partial<User> = {
        id: 1,
        name: "Bob"
    }
    ```
    
- admin은 아래 optional 인터페이스를 구현한 것과 같다고 보면 된디ㅏ.
    
    ```java
    interface User {
        id?: number;
        name?: string;
        age?: number;
        gender?: "m" | "f";
    }
    ```
    

## 3. Required<T>

- 모든 속성을 필수로 만든다.
    
    ```java
    interface User {
        id: number;
        name: string;
        age?: number;
    
    }
    
    let admin: Required<User> = {
        id: 1,
        name: "Bob",
        age: 30
    }
    
    ```
    

## 4. Readonly<T>

- 읽기전용으로 만들어서 재할당할 수 없게한다.
    
    ```java
    interface User {
        id: number;
        name: string;
        age?: number;
    
    }
    
    let admin: Readonly<User> = {
        id: 1,
        name: "Bob",
        age: 30
    }
    
    admin.id = 4; // read-only프로퍼티라서 재할당오류가 난다.
    ```
    

## 5. Record<K, T>

- K는 key이고 T는 type이다.
    
    ```java
    type Grade = "1" | "2" | "3" | "4";
    type Score = "A" | "B" | "C" | "D" | "F";
    
    const score: Record<Grade, Score> = {
        1: "A",
        2: "C",
        3: "B",
        4: "D",
    }
    ```
    
- 값이 맞게 작성되었는지 체크할 수 있다.
    
    ```java
    interface User {
        id: number;
        name: string;
        age: number;
    }
    
    function isValid(user: User) {
        const result: Record<keyof User, boolean> = { // key는 User에 작성된 것으로 되었는지
            id: user.id > 0,
            name: user.name !== '',
            age : user.age > 0,
        };
        return result;
    }
    ```
    

## 6. Pick<T, K>

- 특정 프로퍼티만 사용할 수 있다.
    
    ```java
    interface User {
        id: number;
        name: string;
        age: number;
        gender: "M" | "W";
    }
    
    const admin: Pick<User, "id" | "name"> = {
        id:0,
        name: "Bob",
    }
    ```
    
    - id, name만 가져다가 쓸수 있다.

## 7. Omit<T, K>

- pick과 반대로 특정 프로퍼티를 생략해서 사용할 수 있다.
    
    ```java
    interface User {
        id: number;
        name: string;
        age: number;
        gender: "M" | "W";
    }
    
    const admin: Omit<User, "age" | "gender"> = {
        id:0,
        name: "Bob",
    }
    ```
    
    - User에서 age와 gender를 제외하고 선언할 수 있다

## 8. Exclude<T1, T2>

- T1에서 T2를 제외한다.
- omit은 프로퍼티로 제외하고, exclude는 타입으로 제외시킨다.
    
    ```java
    type T1 = string | number | boolean;
    type T2 = Exclude<T1, number | string>; // T2는 boolean타입만 가능하게 된다.
    ```
    

## 9. NonNullable<Type>

- 널을 제외한 타입을 생성한다.
    
    ```java
    type T1 = string | null | undefined | void ;
    type T2 = NonNullable<T1>; // string, void만 남는다.
    ```
    

# 후기

- 타입스크립트에 반해버렷다.
- 마소가 만든 오픈소스라고 한다. ([깃헙 주소](https://github.com/microsoft/TypeScript))
- 언젠가는 오픈소스 이슈를 해결할 수 있길…!!