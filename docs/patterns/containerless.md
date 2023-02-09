---
layout: post
title: what is containerless
date: 2023-02-09 13:07:45
last_modified_at : 2023-02-09 13:07:45
parent: Patterns
has_children: false
nav_exclude: true
---

토비의 스프링 부트 강의가 인프런에 올라왔네요 !!

강의 내용 중 containerless 아키텍처에 대해 말씀하셔서 추가로 더 정리했습니다.

이렇게 보면 MSA는 선택이 아니라 필수인 것 같습니다.ㅎㅎ

# what is containerless?

- serverless와 유사한 개념으로 서버와 같은 인프라에 관여하지 않고 개발자들이 서비스에 집중할 수 있도록 하는 것

# containerless가 나타난 배경

- what is container?
    - web client가 web request를 주고 web component가 web response를 내보내는 구조에서
    - web component(java의 servlet)는 web container(servlet container로 tomcat..)안에 존재해서 컨테이너에서는 life cycle을 관리하게 된다.
    - 이 때 web container는 하나 이상의 web component를 갖게되는데, client로 들어온 요청을 어느 component가 담당할지 연결해주는 handler mapping 작업 등을 하게 된다.
- spring container의 경우
    - servlet container없이 안되는데, servlet container 뒤쪽에 존재하면서 servlet container로부터 전달받은 요청을 bean에게 전달하는 방식이다.
    - 이렇게 통신하려면 servlet container를 띄워야하고, 그러기 위해선 web.xml로 서블릿 클래스도 연결하고 war로 만들어서 올려야하고 배포하고 등등 설정이 복잡하게 된다.
- 이러한 수고를 제거하기 위한 containerless가 고민된 것으로
    - 이말은 즉슨 containerless를 통해 각각의 서비스들을 나눠서 containerized시키고 orchestrated화하기 위함을 목적이라 할 수 있겠다.
    - 여러개의 서비스를 한개의 container에 넣게 되면, 어찌되었던 다운타임이 발생할 수 있고 이것은 큰 비용으로 이어질 수 있다. 특히 이러한 구조는 cloud환경에서 중요한 사항이다.

## 그래서 containerless하려면?

- 예를 들어 Kubernetes에서 서비스를 올리게 되면, 리소스나 네트워크와 같은 상세 정보를 알지 못해도 Kubernetes 환경이 가진 이점을 그대로 사용할 수 있다.
- spring 또한 이전의 web.xml에 서블릿 컨테이너를 올렸던 대신, spring boot를 통해 실현할 수 있다고 볼 수 있다.

## containerless의 단점은?

- 추상적인 이야기이지만, 표준화되도록 만들어야한다는 것. 안그러면 결국 벤더에 종속하게 된다는 점이 있다.