---
layout: post
title: Why bdd?
date: 2024-05-08 21:30:00
last_modified_at : 2024-05-08 21:30:00
parent: TestCase
grand_parent: Quality Practice
nav_exclude: true
---

# To-do

- 왜 cucumber를 사용하려고 하는지, 이점이 뭔지 파악하기
- spring-context 하위에 작성하기
- jenkins와 통합해서 테스트 자동화 작업
    - JenkinsFile에 cucumber 테스트 단계 추가하고,
    - 리포트 단계도 추가한다.

# Background

- 이미 junit5로 인터페이스 기반 테스트 케이스가 작성되어있는 상태
- API 인수테스트는 karate로 시나리오가 작성되어 일배치로 자동화되어있다.

# Why BDD?

- 그렇다면 추가적으로 왜 bdd를 적용해야하는가?에 대한 설명이 필요했다.
    - 서드파티 API에 대한 테스트 시나리오가 존재하지 않아서, 항상 수동으로 체크하고 있었다.
    - 그래서 서드파티 API를 개발하지 않으면, 비즈니스 도메인을 쉽게 이해하기 어려운 점이 있었다. (학습비용 큼)

# karate vs cucumber

- cucumber는 spring context와의 통합을 지원하고 있다. (karate는 명시적이진 않지만 주입시켜서 동작하게 할 수 있긴하다.)
- cucumber가 자연어 스타일의 특수 구문을 제공하기 때문에, 비개발자도 시나리오를 알 수 있다.
- karate는 rest api 테스트에 특수되어있다. cucumber는 어떠한 종류의 테스트도 가능하기 때문에 인수테스트로도 커버가 가능하다. → 내가 하려는 목적
- cucumber는 junit과 통합이 가능하다.
- cucumber는 Gherkin 언어를 사용하기 때문에 비 개발자도 쉽게 이해할 수 있으며, 테스트 코드와 도메인 언어 간의 분리로 코드 유지보수 및 확장이 편리하다.

# try1. spring context + junit5 + cucumber

- 이 내용도 참고하자. API테스트에 cucumber를 사용하면 안되는 이유
    - [https://m.blog.naver.com/genycho/221600897352](https://m.blog.naver.com/genycho/221600897352){:target="_blank"}
- 만약에 작성한다면, 아래 내용 필독하자
    - [https://rowdie.tistory.com/33](https://rowdie.tistory.com/33){:target="_blank"}
    - [https://github.com/ohjuntaek/lab-cucumber.git](https://github.com/ohjuntaek/lab-cucumber.git){:target="_blank"}

# try2. bddmockito 사용

- 꼭 cucumber로 작성하지 않아도, junit내에서 bddmockito api를 사용해서 Given ⇒ When ⇒ Then으로 작성하는 것으로 시나리오를 만들 수 있다.
- [https://tecoble.techcourse.co.kr/post/2020-09-29-compare-mockito-bddmockito/](https://tecoble.techcourse.co.kr/post/2020-09-29-compare-mockito-bddmockito/){:target="_blank"}

# Conclusion

- 결론은, 좀더 bdd스타일을 적용해서 junit테스트케이스에 bddmockito를 적용하고 잘 사용해보는 것으로 결정했다. 🤣

# Reference

- [https://www.baeldung.com/cucumber-spring-integration](https://www.baeldung.com/cucumber-spring-integration){:target="_blank"}
- [https://www.baeldung.com/cucumber-data-tables](https://www.baeldung.com/cucumber-data-tables){:target="_blank"}
- [https://rowdie.tistory.com/33](https://rowdie.tistory.com/33){:target="_blank"}