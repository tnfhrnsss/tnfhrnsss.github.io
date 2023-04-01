---
layout: post
title: 12factor application + alpha
date: 2023-04-01 23:01:00
last_modified_at : 2023-04-01 23:01:00
has_children: false
parent: Clipping
nav_exclude: true
---

# Twelve Factor 방법론

## background

[https://12factor.net/ko/](https://12factor.net/ko/) 에 따르면, 

```java
It is a triangulation on ideal practices for 
app development, paying particular attention 
to the dynamics of the organic growth of an app over time,
the dynamics of collaboration between developers working 
on the app’s codebase, 
and avoiding the cost of software erosion.

시간이 지나면서 앱이 유기적으로 성장하는 부분, 
앱 코드베이스에서 작업하는 개발자들 간의 협업, 
시간이 지나면서 망가지는 소프트웨어 유지비용을 줄이는 법에 집중하여 
이상적인 앱 개발 방법을 찾고자 했다.
```

배경은 이렇다고 한다. 

요점은 현대 사용되는 어플리케이션들의 문제를 추출하고 개념적으로 방법을 제공해주는 방식인데, 모든 상황에 통용될 수 있는 방법론이라고 볼수 있다.

- msa아키텍처를 갖고
    - api 및 이벤트 기반 서비스로 구조화
- 서버리스 기반의 모델을 갖고
    - 비지니스 로직에만 집중할 수 있도록
- 데브옵스 자동화 ci/cd를 해야한다
    - 코드 기반 배포 및 모니터링 자동화가 되도록

## Factors

### 1. **Codebase**

버전 관리를 할 수 있는 코드베이스를 만들어서 다양한 버전관리에 활용하라

### 2. Dependencies

라이브러리 의존성을 명시적으로 분리하라.

node의 package.json이나 docker의 dockerfile에서처럼 라이브러리의 버전을 명시적으로 선언하는 것처럼

### 3. Config

서버주소나 비밀번호와 같은 환경변수값을 코드에 넣지 않고 별도로 빼서 관리해야한다.

ex) aws lambda variables 서버에서 환경변수 key-value로 저장하고 serverless system manager를 통해서 parameter store를 구성하고 api를 통해 람다에 저장한 값을 조회한다.

### 4. Backend Services

db, 캐시, 메시지큐, 외부 api서버 등을 다른 api 서비스와 구분없이 하나의 “서비스”로 인식해서 관리하라는 것

### 5. Build, release, run

어플리케이션 빌드, 릴리즈, 실행 단계를 분리해서 관리한다. ci cd 파이프라인을 구성하라는 것

- 개발자가 신규 코드를 작성하면 빌드를 통해
- 주기적으로 릴리즈하고, 서버 재시작을 시키고 충돌 발생하면
- 타임스탬프 및 릴리스 id를 기반으로 롤백도 가능하게 해야한다

### 6. Processes

어플리케이션을 독립적인 무상태(stateless)프로세스로 실행해야한다.

cookie나 sticky session등이 담겨있으면 stateful 한 상태로 독립적인 프로세스 상태가 아니다.

캐시같은 것도 db처럼 별도의 독립적인 프로세스로 구성하라는 것

### 7. Port binding

독립적인 포트를 바인딩해서 서비스로 공개하고, 포트를 통해 접근하도록 해서 그 요청을 처리하는 것을 말한다.

- 서버리스의 경우는 포트바인딩 대신에 동기호출은 API로, 큐가 들어온다거나 파일이 업로드 되는 등은 이벤트 기반으로 해서 비동기호출을 받아 동작시킨다거나 또는 poll-based(db에 변화(값이 변화하거나) 발생한것을 감지해서)기반으로 동작시킨다.

### 8. Concurrency

프로세스 모델을 기반으로 scale-up 또는 scale-out확장을 가능하게 하는 것

### 9. Disposability

빠른 시작과 gracefull shutdown을 제공해서 서비서 안정성을 제공해야한다.

### 10. Dev/prod parity

개발, 테스트 및 운영 환경을 최대한 동일하게 유지할 수 있도록 해야한다.

### 11. Logs

로그를 파일로 쌓지말고 어플리케이션이 실행되는 도중에 실시간으로 스트림 방식 등으로 전송되도록 한다.

### 12. Admin processes

백오피스 업무는 기존 어플리케이션과 같은 개발 환경에서 동일 형태로 수행되도록 한다.

### 13. Telemetry

데이터를 어떻게 모니터링하고 관제할 것인가

### 14. Automation

모든 작업을 자동화해야한다.

### 15. Security

어플리케이션 보안 중요하다. 

# Reference

[https://hhjeong.tistory.com/183](https://hhjeong.tistory.com/183)

[https://12factor.net/ko/](https://12factor.net/ko/)

[2226-msa-12-factors-application](https://sarc.io/index.php/cloud/2226-msa-12-factors-application)

[https://youtu.be/mTbS1ddjTE0](https://youtu.be/mTbS1ddjTE0)