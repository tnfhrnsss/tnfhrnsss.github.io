---
layout: post
title: Intercepting Filter Pattern
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Patterns
has_children: false
nav_order: 9
nav_exclude: true
---

# Intercepting Filter Pattern

Interception Filter Pattern

- 체인으로 묶인 필터를 통과하면서 최종적으로 Target 클래스에 도달하도록 한다.
- 전처리/후처리를 수행하려고 하는 경우 사용

## Entity.

### Filter::

실제로 필터작업을 구현하고, 실행

```java
public class AppChannelFilter implements BusinessMessageFilter {  // 특정 App채널만 유효하도록 필터링
    private final AppChannelTypeValidator appChannelTypeValidator;

    @Override
    public void execute(ConversationMessage conversationMessage) {
        appChannelTypeValidator.validate();
    }
}
```

### Filter Chain::

여러 필터들을 순서대로 실행하기 위해 체인으로 묶음

### Target::

모든 필터가 통과되면 최종적으로 실행될 엔티티

### Filter Manager::

여러 필터들과 필터 체인을 연결

```java
public class BusinessMessageFilterManager {
    private final List<BusinessMessageFilter> businessMessageFilters;

    public void execute(ConversationMessage conversationMessage){
        for (BusinessMessageFilter businessMessageFilter : businessMessageFilters) {
            businessMessageFilter.execute(conversationMessage);
        }
    }
}
```

### Client::

필터요청하는 엔티티

```java
public class BtalkMessageFilterExecutor {
    private final BusinessMessageFilterManager businessMessageFilterManager;

    public void filter(ConversationMessage conversationMessage) {
        businessMessageFilterManager.execute(conversationMessage);
    }
}
```

### 참고.

[https://www.tutorialspoint.com/design_pattern/intercepting_filter_pattern.htm](https://www.tutorialspoint.com/design_pattern/intercepting_filter_pattern.htm)