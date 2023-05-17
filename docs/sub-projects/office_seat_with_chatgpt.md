---
layout: post
title: 자리배치도 전산화 작업 with chat gpt
date: 2023-04-10 01:12:00
last_modified_at : 2023-05-18 01:12:00
parent: Sub Projects
has_children: false
nav_order: 7
nav_exclude: true
---

# Goal

- 엑셀로 관리하던 것을 자리배치도를 전산화하고자 함
- 요구사항 리뷰, 모델링, 프로그래밍, 리팩토링까지 모두 chatgpt로 진행하는 것

# 요구사항 정의

- 먼저 무엇을 만들 것인지에 대해 먼저 설명합니다.
    
    ![officeseat_gpt1.png](../img/officeseat_gpt1.png)

    
- 요구사항을 디테일하게 정의하고, 필요하다면 범위를 좁히면 좀더 정확한 결과를 받을 수 있습니다.

![officeseat_gpt2.png](../img/officeseat_gpt2.png)

# 기술 스택 정의

- 작성할 언어를 선택하고, 레이어나 네트워크 설계도 할 수 있고, 프레임워크도 정할 수 있습니다.
- 저는 자바, 스프링 프레임워크, H2, vuejs 정도 사용하는 것으로 했습니다.

![officeseat_gpt3.png](../img/officeseat_gpt3.png)

# 서버 API 작성

- 우선 자리를 지정하는 API부터 작성 요청했습니다.

![officeseat_gpt4.png](../img/officeseat_gpt4.png)

- 컨트롤러-서비스-스토어 영역까지 한번에 작성합니다 ㅎㅎ
- 보면 제가 요청한 것은 자리를 지정하는 API였는데, 해제하는 API까지 작성하고, 사원이 퇴사했을 경우에 대한 공석 처리 요구사항도 코드에 포함되어있습니다.
- 도메인 클래스 및 jpa 엔티티 클래스도 포함되어있습니다.
- 코드에 대해 리팩토링을 요청하면, 다시 모델링을 거친 모습으로 코드를 작성합니다.
    - 인터페이스로 추출을 요청하니, 맞는 설계라며 다시 코드를 작성해주기도 하네요
    - 도메인 객체에 setter를 안써야하지 않냐고 질문을 했었는데요.
        
        ![officeseat_gpt5.png](../img/officeseat_gpt5.png)
        
    - 결국 도메인 객체에 작성된 setter는 변경되었지만, 뭔가 짝프로그래밍을 하고 있다는 생각에 이 때부터 재밌어진 것 같습니다.
    - 그래서 집요하게 질문을 좀더 많이 했습니다.
    - API 규약에 대해 질문을 했습니다.

![officeseat_gpt6.png](../img/officeseat_gpt6.png)

- 자리 (Seat)엔티티와 Employee 엔티티는 one to one 관계가 되어야하는데. 자꾸 ManyToOne으로 코드를 작성해서 설명을 하는 과정이 있었습니다. 어쩌면 요구사항 정의 단계에서 잘못 전달되었을지 모르겠네요.

# 후기

- 질문은 순서대로 하는 것이 좋습니다. 요구사항 정의부터 하나씩 대화하면서 정리해야 합니다.
    - 첨엔 그냥 요구사항 말하면 코드를 한번에 다 알려주는 줄 알았는데요. “제가 코드를 직접 작성해드릴 수는 없습니다.” 이렇게 답변받음
- 마치 동료와 pair 프로그래밍을 하는 것과 같았습니다. 메신저로 동료와 코드리뷰하듯이 대화를 나눌 수 있습니다.
- 혼자서 유튜브나 책보고 공부하는 것과 또 다른 경험인 것 같습니다. 왜 이렇게 코딩했는지 질문하면 상세하게 이유도 설명해주고. 실시간 리뷰를 주고받으며 코딩하는 것이 꽤나 재밌습니다!!
- 실시간 처리를 요하는 비지니스를 처리하기에는 아직 무리가 있습니다.(정확한 답을 요할수록 딜레이가 있습니다.)
- 코드리뷰에도 도움이 되고, 의외의 인사이트도 얻을 수 있습니다.
- 한글보다 영어로 질문하면 더 정확하고 빠른 답변을 얻을 수 있습니다.

# Reference

- 자리배치를 전산화하는 방법은 leaflet을 사용해서 하는 방법도 있다고 합니다.
    - [floor-map-management-system-on-web-with-leaflet/](https://engineering.linecorp.com/ko/blog/floor-map-management-system-on-web-with-leaflet/)
    - [지도용-라이브러리로-좌석배치도-만들기](https://velog.io/@brviolet/%EC%A7%80%EB%8F%84%EC%9A%A9-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC%EB%A1%9C-%EC%A2%8C%EC%84%9D%EB%B0%B0%EC%B9%98%EB%8F%84-%EB%A7%8C%EB%93%A4%EA%B8%B0)
     
- vuejs랑 spring boot 서버랑 동일 프로젝트로 생성  
    - [https://jiurinie.tistory.com/102](https://jiurinie.tistory.com/102)  

- replit에서 chatgpt에 질의하면 만든 TechCo 사이트 제작 동영상입니다.(https://en.rattibha.com/thread/1599803817515548674)  