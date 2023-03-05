---
layout: post
title: 토비의 봄 TV 스프링 리액티브 프로그래밍(1) - Reactive Streams
date: 2023-03-05 20:12:00
last_modified_at : 2023-03-05 20:12:00
parent: Youtube
grand_parent: Mooc
nav_exclude: true
---

## 강의 주소

[토비의 봄 TV 스프링 리액티브 프로그래밍(1)](https://www.youtube.com/watch?v=8fenTR3KOJo)

## 요점

1. collection이 아닌 iterable도 for-each로 쓸 수 있다
    
    ```kotlin
    Iterable<Interger> iter = Arrays.asList(1,2,3,4,5,6);
    for (Interger num : iter) {
    	System.out.println(num);
    }
    ```
    
    - 익명 클래스 보단, 람다식으로 변경
        - 메서드명 필요없고
        - 접근제어자 필요없고
        - 리턴값 필요없고
        
        ```java
        Iterable<Integer> iter = () -> {
        	return null;
        };
        
        //
        Iterable<Integer> iter = () -> 
        	new Interator<>() {
        	   @Override
        			public boolean hasNext() {
        				return false;
        			}		
        
        		@Override
        	public Integer next() {
        			return null'
        		}
        
        	};
        };
        ```
        
    - Iterable vs Observable (duality) - 에릭 마이어로 검색해볼 것
        - iterable은 pull방식으로 next로 가져오는데 observable은 push
        - Observable api
            - java9에선 deprecated됨
            - source에서 event를 observer로 던지는데, noti가 올때마다 던지는
                - Observable : source → event/data → Observer
            - 여기서 나온게 observer 패턴
            
            ```java
            // publisher
            static class IntObservable extends Observable implements Runnable  {
            	@Override
            	public void run() {
            		for (int i=1; i<=10; i++) {
            			setChanged();  // 상태 변경하고
            			notifyObservers(i);  // 알린다(push). iterable로 치자면 next()함수가 되겠다. next는 pull
            		}
            	}
            }
            
            // subscriber
            public static void main(String[] args) {
            	Observer ob = new Observer() {
            		@Override
            		public void update(Observable o, Object arg) {
            			System.out.println(arg);
            		}
            	};
            
            	// Observable이 던지는 이벤트를 다 받게 된다. 
            	IntObservable io = new IntObservable();
            	io.addObserver(ob);
            	io.run();
            }
            ```
            
    
    1. Reactive Streams
    - 배경
        - ReactvieX를 개발한 마소 개발자들이 Observer 패턴에 대해 지적한 단점
            1. Observer에 던진 데이터가 마지막 데이터라는 것을 알 수 없다. Complete에 대해 알 수 없음
            2. Exception발생했을 때, 예외가 전파되는 방식이나 fallback에 대해 구현한다거나에 대한 것이 없다.
            3. 그래서 나온 확장된 Observer 패턴으로 reactive streams
    
    ```java
    A Publisher is a provider of a potentially unbounded number of sequenced elements, 
    publishing them according to the demand received from its Subscriber(s).
    
    In response to a call to Publisher.subscribe(Subscriber) the possible invocation 
    sequences for methods on the Subscriber are given by the following protocol:
    
    onSubscribe onNext* (onError | onComplete)?
    ```
    
    - Publisher는 연속적인 요소의 한계가 없고, Subscriber가 받을 수 있게
    - 프로토콜 : onSubscribe를 반드시 호출해야하고 onNext는 한번도 안해도 되고 여러번 호출해도 된다.
    - reactive stream과 java9의 Flow stream같다.
    
    ```java
    Iterable<Integer> iter = Arrays.asList(1,2,3,4,5,6,7);
    Publisher p = new Publisher() {
    	@Override
    	public void subscribe(Subscriber subscriber) { // 이건 Publisher가 subscribe()를 제공하는 것이고 Subscriber가 이걸 호출하는 것
    		subscriber.onSubscribe(new Subscription() {
				@Override
				public void request(long n) {
					// 이걸 요청했더라도 publisher에 데이터가 없을 수 있다.
					while(true) {
						if (it.hasNext()) {
							subsriber.onNext(it.next));
						} else {
							subscriber.onComplete();
							break;
						}
					}
				}
			
				@Override
				public void cancel() {
						
				}
		});
    }
}
    
    Subscriber<Integer> s = new Subscriber<Integer>() {
    	Subscription subscription;
    
    	@Override
    	public void onSubscribe(Subscription subscription) {
    		this.subscription = subscription;
    		this.subscription.request(Long.MAX_VALUE);
    	}
    
    	@Override
    	public void onNext(Integer item) { // observer패턴의 update와 같다. 다음꺼 줬으니 처리하라는 것
    		this.subscription.request(1);
    	}
    
    	@Override
    	public void onError(Throwable throwable) { // 익셉션을 호출하지 말고 subscription이후에 발생하는 에러에 대해서 전담
    		
    	}
    
    	@Override
    	public void onComplete() { // 더이상 줄것이 없다는 것
    		
    	}
    
    }
    ```
    
    ![tobylee_reactivestream1](../img/tobylee_reactivestream1.png)
    
    - Subscription은 중개역할로 이걸 통해서 요청할 수 있다.
        - request 함수 : backpressure 역할로 publisher와 subscriber사이에 속도차가 발생했을 경우에 그걸 조절하는 역할을 한다.

# Reference

[reactive-streams-jvm github](https://github.com/reactive-streams/reactive-streams-jvm)