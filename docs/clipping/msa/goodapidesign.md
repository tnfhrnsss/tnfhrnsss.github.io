---
layout: post
title: 좋은 API 디자인이란
description: about api design
date: 2023-08-14 10:04:00
last_modified_at : 2023-08-14 10:04:00
has_children: false
parent: Clipping
nav_exclude: true
---

누구에게 좋은 API 디자인인지 고민해봐야합니다.

- 클라이언트 개발자
- 다른 연동 개발자
- 품질팀
- 기획팀
- 나 자신

나 자신조차 작성한 API의 요구사항에 대해 헷갈려한다면 잘 만든 API라고 볼 수 없습니다.

“[일상 속 사물이 알려주는 웹API 디자인](http://www.yes24.com/Product/Goods/94462254)” 책을 중심으로 학습하고 

API를 설계할 때마다 참고하고자 정리한 내용입니다.

# Check List

## 1. 사용자를 위한 API 디자인하기

- 경로를 포함한 리소스 표현
    - url은 명확해야한다.
    - REST 리소스경로는 유일해야한다 → 유일성을 위해서 사용자 편의성(명료성)을 없애면 안된다. (ex. /c → /catalog)
    - 일반적인 포맷은 /{콜렉션의 아이템의 타입을 반영하는 복수형 이름}/{아이템의 id}
- 컨셉 디자인
    - 반드시 각 속성이 컨슈머 관점으로 디자인되도록 한다.
    - 개별 속성을 컨슈머가 이해할 수 있는지
    - 컨슈머가 관심을 두는 속성인지
    - 컨슈머에게 쓸모있는 정보인지
    - 컨셉에서 response 디자인 - 하나의 컨셉에서 서로 다른 response 디자인
    - 컨셉과 response에서 파라미터 디자인
    - 파라미터는 반드시 필요한 데이터만 제공해야하고 불필요한 정보는 제공하지 않는다.
    - 백엔드가 자체적으로 처리할 수 있는 데이터를 포함해서도 안된다.
- 사용자가 할 수 있는 일에 집중하면 인터페이스는 단순해진다.
- API를 통해 “무엇을” “어떻게” 하는지가 곧 API 목표가 된다.

## 직관적인 API 디자인하기

1. 상태가 없는 흐름 디자인하기
    1. 상태없음은 서버와 리퀘스트 사이에 세션 등을 통해 중간에 저장하는 컨텍스트가 존재하지 않아야 한다는 것을 의미한다. 리퀘스트는 오직 리퀘스트가 제공하는 정보를 통해서만 처리되어야 한다.
    2. API목표를 독립적으로 사용할 수 있게 만들어야 다른 컨텍스트에서도 API를 재사용할 수 있다.

## 프로그래밍 인터페이스 디자인하기

- 리소스 경로는 반드시 사용자 친화적이어야 한다.
- patch vs put
    - patch : 리소스 부분 정보를 수정
    - put : 리소스 교체, 만약에 리소스가 없다면 새로 생성해서 반환한다.
- post는 유즈케이스에 적합한 다른 메서드가 없을 때 기본값으로 택하는 메서드다.
- 데이터를 디자인할 때는 반드시 object, array, string, number, date, boolean 타입같은 포터블 데이터 타입을 사용해야 한다.

## 예측 가능한 API 디자인하기

1. 일관된 데이터 디자인하기
    1. 일관성 있는 이름과 일관성 없는 이름
    2. 일관성 있는 데이터 타입과 일관성이 없는 데이터 타입과 포맷
    3. 일관성 없는 URL 구조
    4. 일관적이지 않은 데이터 구조(데이터 타입, 포맷, 조직화)
2. 4단계 일관성
    1. 1단계 - API내부에 일관성이 존재
    2. 2단계 - 팀API / 회사 / 조직 간의 일관성이 존재
    3. 3단계 - 도메인 간의 API의 일관성이 존재
    4. 4단계 - 외부 세계와 일관성이 존재
3. 요약
    1. 일관된 규칙을 정의하고 표준을 준수해서 동작을 추측할 수 있는 API를 작성해야한다.

## 간결하고 체계적인 API 디자인하기

1. 데이터 구조화하기
    1. 연관성 표현하는 방법
        1. 필드 순서 위치를 통한 연관성 부여 : 연관성이 있으면 순차로 기술
        2. 필드 이름을 통한 연관성 부여 : 연관성이 있으면 동일한 단어로 시작하게 한다
        3. 위치와 이름을 통한 연관성 부여
        4. 하위 구조를 통합 연관성 부여
            
            ```java
            {
            	"overdraftProtection" : {
            			"active": true,
            			"limit": 100
            	}
            }
            ```
            
    2. 정렬 순서로 중요도 표현
2. API 사이징
    1. 데이터를 세분화하더라도 속성의 깊이는 최대 3단계까지 구성하도록 추천한다.
    2. 속성의 개수도 가능한 최소한으로 구성한다. 
        1. 속성이 20개가 넘어가면 재검토가 필요하다.
3. 목표 세분화 선택하기
    1. 올바른 세분화를 하려면 API의 목표가 이질적인 2가지 이상을 갖고 있으면 안된다. 다른 목표가 숨겨져 있음을 알기 어렵고
    2. 모든 걸 다 하는 목표는 좋지않다

## 상황에 맞는 API 디자인

1. 안전한 API 디자인하기
    1. 단순하지만 더 굵직한 스코프로 정의하기
        1. 카테고리 기반으로 스코프로 정하는 것은 좋은 생각이 아니다 - 자칫 비대해짐
        2. 역할과 기능성 기반의 스코프로 정의합니다.

## 체크리스트

### 요구사항 분석하기

| 무엇을 하고 싶은지, 어떤 상황인지 | 요구사항 개요를 파악하고, 컨텍스트를 파악하기 위해 제일 먼저 묻는 것 |
| --- | --- |
| 어떻게 쓰일 것인지 | 컨텍스트에 대한 자세한 정보를 제공한다 |
| 컨슈머와 최종 사용자는 누가 될 것인지 | 디자인의 방향성을 제시해 준다 |
| 5 why 분석법 | “문제”의 근본 원인을 식별해냅니다.  |
| API 목표 캔버스 그리기 |  |

### 컨슈머의 컨텍스트 조사하기

| 컨슈머들에게 제약사항이 있는지 | 컨슈머가 xml만 사용가능할 수도 있고, http patch는 지원하지 않을 수 있다 |
| --- | --- |
| 업계 표준이 있는지 |  |
| 그들이 어떠한 네트워크 환경에 속해 있는지 | API 환경에 따라 API를 디자인할 때 약간의 주의가 필요할 수 있다. |

### 프로바이더의 컨텍스트 조사하기

| 뒷단에는 어떤 시스템/API/팀/파트너가 있는가? | 의존성을 늦게 식별하면 제약이 생기는 상황이 발생하기 때문에 미리 식별 필요.  |
| --- | --- |
| 기술적인 제약사항이 있는지 | 24/7 실행이 불가능하거나 , 비동기만 가능하거나, 처리/응답 시간이 길거나, 확장이 불가능하다거나 |
|  기능적인 제약사항이 있는지 | 호환되지 않는 기존 비즈니스 규칙이 있는지 |
| 사람이 처리해야 하는 부분이 있는지 | 뒷단에 사람이 개입하는 절차가 있는지 |
|  어떤 종류의 통신이 필요한지 | 동기화, 비동기, 스트림, 이벤트 등등 인지 식별필요 |
| 어떤 타입의 API가 필요한지 | 요구사항과 컨텍스트에 따라 필요한 API타입이 무엇인지 식별한다 |
| API가 솔루션이 맞는지 | 요구사항과 컨텍스트에 따라 필요한 API타입이 무엇인지 식별한다 |

### 보안 가능성 조사하기

- API가 민감 요소를 취급하는가?

### API 린트 체크리스트

| 카테고리화 | 목표를 분류하면 API를 쉽게 이해할 수 있다 |
| --- | --- |
| path 적합성 | 경로 포맷은 지침을 준수하고, 기존 구성요소와 일관성을 지녀야한다. 여기에는 성공 response 타입도 포함된다. |
| http 메서드 적합성 | 지침을 준수하고, 수행해야하는 목표에 적합한지 |
| 네이밍 적합성 | 파라미터 이름이 지침을 준수했는지, 기존 구성요소와 일관성이 있어야한다. |
| 필수 여부 적합성 | 파라미터의 선택/필수 상태는 지침을 준수했는지, 경로 파라미터는 필수요소로 되어야 한다는 것. 모든 바디 파라미터가 선택사항이면 생성/변경 시에 에러 유발 |
| 타입과 포맷 적합성 | 예: 쿼리 파라미터에 오브젝트는 사용할 수 없고 url에 보이지 않는 문자도 사용할 수 없다. |
| 위치 적합성 | http 상태 메서드를 준수해야한다. 예: 바디 파라미터는 delete 리퀘스트에서 사용할 수 없다. |
| 제공 가능 여부 | 컨슈머가 각 목표에 이 파라미터들을 제공할 수 있어야 한다. |
| http 상태, body, header 적합성 | response의 http상태는 지침을 준수하고 response타입에 적합해야한다.(성공, 컨슈머 실패, 프로바이더 실패) |
| 사용 여부 | response로 내려가는 데이터가 실제로 쓰이는지 확인해야한다. |
| 브레이킹 체인지 포함여부 | 브레이킹 체인지를 유발할 경우, 이게 꼭 필요한지 리뷰해야한다. 해야할 경우 버전 정책을 적용해야한다. |
| 모델 사용여부 | 각 모델 정의는 반드시 사용되어야 한다. 만약 정의된 내용이 사용되지 않는다면, 목표에서 적용하는 걸 망각했는지 확인해야 한다. 그것도 아니라면 제거해야 한다 |
| 모델 이름, 모델 데이터 구조, 설명 적합성 | 지침 준수하고 기존 구성요소와 일관성이 있는지 |
| 모델의 속성 이름, 필수여부, 타입과 모포맷, 설명 적합성 | 지침을 준수하고 기존 구성요소와 일관성을 지녀야 한다. |

# 상황에 맞는 API 디자인

## 컨텍스트에 맞는 API 디자인하기

1. 처리 시간이 오래 걸리는 작업에 대한 API 디자인 방식
    1. 비동기 방식으로 한다(http status 202)
        1. 컨슈머에 이벤트 알리기 = webhook
    2. 이벤트 흐름 스트리밍하기
        
        ```java
        GET /stocks/APL/prices
        Accept: text/event-stream
        ```
        
        1. 컨슈머가 읽고 처리하는 사이에 발생하는 변경을 놓칠 수 있기 때문에, **Server Sent Events(SSE)**를 이용해서 컨슈머가 stream을 요청하고 서버에서 목록형태로 반환하는 방식
        2. 새로운 이벤트가 더 이상 발생하지 않거나 컨슈머가 연결을 닫기 전까지 지속적으로 발생한다.
        3. webhook을 써도 되지만, 모든 데이터 변동에 대해 모든 컨슈머에게 항상 보내고 있어야 한다는 이슈가 있다.
        4. 단점: http연결이 오랜 시간 동안 열려있을 수 있어서 성능 이슈가 없도록 튜닝 필요하다
        
    3. websocket 사용
        1. 이 방법은 sse스트림보다 인프라에서 해야할 것이 많다. 프록시도 구성해야하고

2. API를 디자인할 때, 리퀘스트 전후로 실제로 무슨 일이 벌어지는지 깊이 알고 있어야만 기능적 제약사항이나 기술적 제약사항을 포착할 수 있습니다.

🚧 프로토콜의 규칙을 엄수해야 일관된 API를 만들 수 있다!

🚧 API를 디자인할 때 내면의 열정이나 개인의 취향을 억눌러야 한다. 이는 API가 해결해야 하는 모든 상황에 대해 이상적인 해법이 될 수 없다.

## Conclusion

좋은 API를 디자인하는 방법은 최종적으로 정리했을 때

- 철저한 요구사항 분석
    - 인터페이스를 둘러싼 전체 맥락을 파악
    - 모든 사용자와 모든 컨텍스트에 관련된 사항에 공감이 되어야 한다.
- 반드시 커스터머의 측면과 프로파이더 측면 무도 구려해야 한다.
- 개발자 경험 (DX) 체크를 하자 - API문서화, 기술 지원 등

41page

- (어떤 형태이든) 시스템은 개발한 조직의 의사소통 구조를 닮아간다.(Mel Conway)

## reference

- API 스타일북

[API Stylebook](http://apistylebook.com/)

- [https://research.google/pubs/pub46310/](https://research.google/pubs/pub46310/) - rest 단점과 파생 아키텍처 스타일