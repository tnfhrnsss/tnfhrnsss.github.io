---
layout: post
title: AngularJS Advanced Framework Techniques
date: 2018-07-09 13:07:45
last_modified_at : 2018-07-09 13:07:45
parent: Etc
grand_parent: Mooc
nav_exclude: true
---

# [EDX] AngularJS 수강 후 정리

1. 개발 : visual studio code
2. Data Access방법 세가지
    1. Service :You are returning an object with methods.
    2. Factory : The value you are providing needs to be calculated based on other data.
    3. Provider : You want to be able to configure, during the config phase, the object that is going to be created before it’s created. Use the Provider mostly in the app config, before the app has fully initialized.
    4.  세개의 차이점은 복잡도 차이다. Service가 가장 간단하고 Provider가 실시간으로 설정이 가능하다.
3. Service 싱글톤
    1. 싱글톤은 new로 한다.
    2. Angular는 $http같은 서비스를 제공한다. 내장된 서비스들이 $를 붙인다. 그 외에 자체적으로 angular identifier를 만들 수 있다.
        1. [https://github.com/johnpapa/angular-styleguide/tree/master/a1#services](https://github.com/johnpapa/angular-styleguide/tree/master/a1#services)
    3. 싱글톤 서비스 사용하기
        1. 선언한다
            
            ```jsx
            myApp.service('thisService', function(){
            
            })
            ```
            
        2. this를 써서 함수를 set한다
            
            ```jsx
            myApp.service('thisService', function(){
            
            var _food = 'Chicken';
            
            var _drink = 'Soda';
            
            this.getFood = function(){
            
            return _food;
            
            }
            
            this.getDrink = function(){
            
            return _drink;
            
            }
            
            return this;
            
            })
            ```
            
        3. 만약에 새 인스턴스로 생성하고 싶다면
            
            ```jsx
            myApp.service('thisService', function(){
            
            var _food = 'Chicken';
            
            var _drink = 'Soda';
            
            this.getFood = function(){
            
            return _food;
            
            }
            
            this.getDrink = function(){
            
            return _drink;
            
            }
            
            return new this;
            
            })
            ```
            

- -컨트롤러단에 생성한 서비스를 호출하려면, dependency로 선언해야한다.

myApp.controller('myController', ['thisService', function(thisService){

}])

- -서비스를 초기화하려면 new로한다.

myApp.controller('myController', ['thisService', function(thisService){

var serviceInstance = new thisService;

}])

- - 서비스에 선언한 것들에 접근하려면 메소드를 사용한다,

myApp.controller('myController', ['thisService', function(thisService){

var serviceInstance = new thisService;

var someFood = thisService.getFood();

var someDrink = thisService.getDrink();

}])

4. Factory 싱글톤

(1) Service가 new키워드로 생성하는거랑 다르게 Factory는 새 오브젝트 생성전에 조작이 가능하다.

- -Factory선언

myApp.factory('thisFactory', function(){

})

- -factory로 리턴받을 때 속성 지정

myApp.factory('thisFactory', function(){

var factoryObject = {};

factoryObject.food = 'Chicken';

factoryObject.drink = 'Soda';

factoryObject.getAll = function(){

return factoryObject.food + ' ' + factoryObject.drink;

}

return factoryObject;

})

- -새로운 생성자로 리턴받기

myApp.factory('thisFactory', function(){

var factoryObject = {};

factoryObject.food = 'Chicken';

factoryObject.drink = 'Soda';

factoryObject.getAll = function(){

return factoryObject.food + ' ' + factoryObject.drink;

}

return new factoryObject;

})

- -컨트롤러에 factory를 선언할때, dependency로 선언한다

myApp.controller('myController', ['thisFactory', function(thisFactory){

}])

- -factory의 하위 속성에 접근해서 함수를 사용하고자 할떄

myApp.controller('myController', ['thisFactory', function(thisFactory){

var someFood = thisFactory.food;

var someDrink = thisFactory.drink;

var someOfAll = thisFactory.getAll();

}])

## reference

[https://docs.angularjs.org/api/ng](https://docs.angularjs.org/api/ng)