---
layout: post
title: Optional 정리
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Java
has_children: false
nav_order: 2
grand_parent: Clipping
---

# Optional 정리

참조 블로그 : 

[](https://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/)

effective java에서도 말하고 있지만, optional은 API 클라이언트에게 리턴값이 없음을 알려주는 기능에 초점을 맞춰야한다. 검사예외와 비슷한 취지의 쓰임으로 이해하면 된다.

참조 블로그 본문 내용 중

## :: orElse(new ...) 대신 orElseGet(() → new ...)

orElse(new ...) 대신 orElseGet(() → new ...) 를 사용하도록 설명하는 부분이 있다.

이유는 orElse에 있는 값의 존재유무를 불문하고 무조건 실행되기 때문인데.

아래의 경우는 예외이다.

```java
@Slf4j
@RequiredArgsConstructor
@Service
public class KakaoMessageAdapterService {
		private final List<KakaoMessageAdapterHandler> kakaoMessageAdapterHandlers;

		private ThirdPartyMessage getThirdPartyMessage(KakaoMessage kakaoMessage) {
        KakaoMessageAdapterHandler kakaoMessageAdapterHandler = kakaoMessageAdapterHandlers.stream()
            .filter(handler -> handler.filter(kakaoMessage.getType().getThirdPartyMessageType()))
            .findFirst()
            .orElse(kakaoEtcMessageService);

        return kakaoMessageAdapterHandler.send(kakaoMessage);
    }
}
```

- flow
    - thirdparty messsage type별로 KakaoMessageAdapterHandler를 상속받아서 구현체가 존재한다.
    - 만약에 특정 type에 구현체가 존재하지 않는다면, kakaoEtcMessageService가 kakaoMessageAdapterHandler의 구현체가 된다.
    
- 위의 경우 , filter로 선 체크 후 → findFirst하고 있는데 이미 구현체들을 filter로 통과시키기 때문에 무의미하게 kakaoEtcMessageService를 호출하고 있지 않다.

## :: 단지 값을 얻을 목적이라면 Optinal 대신 null 비교

optinal api가 생겼다고, 무조건 wrapping을 한다든지 남용하면 안된다.

optional은 비용이 발생하므로 단순 null체크는 null로 하자.