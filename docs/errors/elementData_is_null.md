---
layout: post
title: Cannot read the array length because "elementData" is null
date: 2023-11-04 23:50:00
last_modified_at : 2023-11-04 23:50:00
parent: Errors
has_children: false
nav_exclude: true
tags: [junit, test case, mock]
description: Cannot read the array length because "elementData" is null
keywords: hyselementDatatrix
--- 

# error log

```prolog
java.lang.NullPointerException: 
Cannot read the array length because "elementData" is null
at spectra.attic.talk.messengerconnector.business.message.send.service.flow.impl.BusinessMessageFlowServiceImplTest.setup(BusinessMessageFlowServiceImplTest.java:57)
```

# problem

- 여러개의 구현체를 갖고 있는 interface를 junit 단위 테스트케이스로 작성하게 되면, 커버를 해야하는 문제가 있다.

# service code

```java
public class BusinessMessageFlowServiceImpl implements BusinessMessageFlowService {
    private final List<BusinessMessageTypeFlowService> businessMessageTypeFlowServices;

    @Override
    public void message(Message message, ConversationBusinessMessage businessMessage) {
        for (BusinessMessageTypeFlowService messageTypeFlowService : businessMessageTypeFlowServices) {
            messageTypeFlowService.send(message, businessMessage);
        }
    }
}
```

- BusinessMessageTypeFlowService 인터페이스를 구현한 모든 서비스에 대해서 테스트 케이스가 필요한 상황으로
- 처음에는 이렇게만 작성했었다
    
    ```java
    @Mock
    private List<BusinessMessageTypeFlowService> businessMessageTypeFlowServices = new ArrayList<>();
    
    @Mock
    private BusinessTextMessageFlowServiceImpl businessTextMessageFlowService;
    
    @Mock
    private BusinessFileMessageFlowServiceImpl businessFileMessageFlowService;
    
    @Mock
    private BusinessTemplateMessageFlowServiceImpl businessTemplateMessageFlowService;
    
    @Before
    public void setup() {
        businessMessageTypeFlowServices.add(businessTextMessageFlowService);
        businessMessageTypeFlowServices.add(businessFileMessageFlowService);
        businessMessageTypeFlowServices.add(businessTemplateMessageFlowService);
    }
    
    @Test
    public void message() {
        businessMessageFlowServiceImpl.message(TextMessage.sample(), ConversationBusinessMessage.sample());
        verify(businessTextMessageFlowService).send(any(Message.class), any(ConversationBusinessMessage.class));
    }
    ```
    

- 결국 setup에서 `elementData` 가 null인 에러발생

# Solve

```java
@Spy
private List<BusinessMessageTypeFlowService> businessMessageTypeFlowServices 
= mock(ArrayList.class, withSettings().useConstructor().defaultAnswer(CALLS_REAL_METHODS));
```

- Mockito를 사용하여 `ArrayList` 클래스의 mock개체를 만드는 것으로 중요한 것은 @Spy를 선언한 것으로 java17 이상이 아니면 Spy를 통해서 mock객체를 만들어야 한다.
    - `useConstructor` 는 `ArrayList` 클래스의 생성자를 호출하여 mock객체를 초기화하라는 의미
    - `CALLS_REAL_METHODS`는 mock 객체가 호출될 때 실제 메서드를 호출하도록 지정하는 것

# Reference

- [https://stackoverflow.com/a/76127813/14257397](https://stackoverflow.com/a/76127813/14257397)
- [https://github.com/mockito/mockito/issues/2589](https://github.com/mockito/mockito/issues/2589)