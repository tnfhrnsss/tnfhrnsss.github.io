---
layout: post
title: Multiple Chain of Responsibility
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Patterns
has_children: false
nav_order: 5
nav_exclude: true
---

역학 사슬 패턴

- 체인에 묶인 객체들끼리 순서대로 책임을 넘기는 구조

## AS-IS.

- 하나의 서비스에서 접수 처리와 메시지 전송 기능이 혼재

## TO-BE.

![Untitled](../img/cor.png)

- 접수 처리 프로세스와 메시지 전송 프로세스를 별개로 분리
- 프로세스 순서를 체인으로 강제 ( 접수 프로세스 진행 후 → 메시지 전송 프로세스 진행)
- 3개 이상의 프로세스 순서를 체인으로 강제하는 것 추가

### :: Client

```java
// 우선순위 talkInitProcessor -> talkAcceptProcessor -> talkJoinForceProcessor -> messageProcessor
public void message(ConversationMessage conversationMessage) {
    talkInitProcessor
        .setNext(talkAcceptProcessor);

    talkAcceptProcessor
        .setNext(talkJoinForceProcessor);

    talkJoinForceProcessor
        .setNext(messageProcessor);

    talkInitProcessor.support(conversationThirdPartyMessage);
}
```

### :: Handler Processor

```java
@Slf4j
public abstract class BtalkProcessor {
    private BtalkProcessor next = null;

    public BtalkProcessor setNext(BtalkProcessor next) {
        this.next = next;
        return next;
    }

    public final void support(ConversationMessage conversationMessage) {
        if (process(conversationMessage)) {
            action(conversationMessage);
        } else if (next != null) {
            next.support(conversationMessage);
        } else {
            log.error("There is no next process of btalk.");
        }
    }

    public abstract boolean process(ConversationMessage conversationMessage);

    public abstract void action(ConversationMessage conversationMessage);
}
```

### :: Sub Processor

```java
[talkInitProcessor]
public class TalkInitProcessor extends BtalkProcessor {
    @Override
    public boolean process(ConversationMessage conversationMessage) {
        return isPresent(conversationMessage);
    }

    @Override
    public void action(ConversationMessage conversationMessage) {
        sendMessage(conversationMessage);
    }
}

[talkAcceptProcessor]
public class TalkAcceptProcessor extends BtalkProcessor {
    @Override
    public boolean process(ConversationMessage conversationMessage) {
        return !isPresent(conversationMessage);
    }

    @Override
    public void action(ConversationMessage conversationMessage) {
        reception(conversationMessage);
    }
}
```

### 참고.

[https://lktprogrammer.tistory.com/45](https://lktprogrammer.tistory.com/45)