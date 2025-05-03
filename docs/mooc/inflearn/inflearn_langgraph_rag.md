---
layout: post
title: AI 에이전트로 구현하는 RAG 시스템(w. LangGraph) 수강 후기
date: 2025-05-04 01:20:00
last_modified_at : 2025-05-04 01:20:00
parent: Inflearn
grand_parent: Mooc
nav_exclude: true
tags: [ai, langgraph, rag]
---

# 강의정보

- [인프런 강의 주소](https://inf.run/5AqAB){:target="_blank"} 
- 강사 : **판다스 스튜디오**
- 총 54개 (6시간 45분)
- 선수지식 : langChain
- 난이도 : 초급~중급
- 수강료 : 유료

# 환경

- python 3.13
- poetry : 가상환경용
    - 설치 및 환경설정
        
        ```java
        curl -sSL https://install.python-poetry.org | python3.13 -
        
        peter@peter ~ % export PATH="/Users/peter/.local/bin:$PATH"
        peter@peter ~ % source ~/.zshrc
        peter@peter ~ % poetry --version
        Poetry (version 2.1.2)
        ```
        
    - certification 관련 에러 참고
- 프로젝트 생성
    
    ```java
    peter@peter _GIT % poetry new langpraph_agent
    Created package langpraph_agent in langpraph_agent
    ```
    
- pyproject.toml 수정
    
    ```java
    [tool.poetry.dependencies]
    python = "^3.11,<3.13.3"
    langchain = "0.3.1"
    langgraph = "0.2.34"
    langchain-openai = "0.2.1"
    langchain-community = "0.3.1"
    langchain-chroma = "0.1.4"
    langchain-ollama = "0.2.0"
    langchain-google-genai = "2.0.0"
    langchain-groq = "0.2.0"
    gradio = "4.44.1"
    python-dotenv = "1.0.1"
    bs4 = "0.0.2"
    wikipedia = "1.4.0"
    ```
    
- 가상환경 생성 및 의존성 패키지 설치
    
    ```java
    poetry env use python3.11 
    poetry install
    ```
    
    - 설치 중 호환이 안되는지 Installing onnxruntime (**1.21.0**): **Failed 실패다.**
        
        Unable to find installation candidates for onnxruntime (1.21.0)
        
        - 일단 Skip!! 결정
- vscode에 poetry 가상환경 커널로 등록
    - 원래 제대로라면 이 과정이 필요없지만, 로컬에 여러 버전의 파이썬을 사용해서 그런지,,과정이 순탄치 않았다. ㅜㅠ
    - poetry shell 설치(최근 몇 버전이후로 shell이 기본설치에서 제외되었기 때문에 별도로 activate시켜줘야한다. 또는 설치)
        
        ```java
        poetry self add poetry-plugin-shell
        ```
        
    - ipykernel 설치
        
        ```java
        peter@peter langpraph_agent % poetry shell
        (langpraph-agent-py3.13) peter@peter langpraph_agent % source $(poetry env info --path)/bin/activate
        (langpraph-agent-py3.13) peter@peter langpraph_agent % pip install ipykernel
        ```
        
    - poetry 커널 생성
        
        ```java
        (langpraph-agent-py3.13) peter@peter langpraph_agent % python -m ipykernel install --user --name=langpraph-agent --display-name "Poetry (langpraph)"
        ```
        
- 프로젝트 셋팅
    - .env 생성 후 openapi key 설정

# Agent란

- langgraph에서 agent는 llm을 기반으로 하며, 특정 역할이나 기능을 수행하도록 설계
- 역할
    1. router
        1. 사용자 질문이 들어왔을때, 분석을 llm이 먼저하고 그 다음 스텝에 대해서 llm이 결정하는 역할
        2. llm이 선택할수 있는 다음 스텝에 대한 옵션을 정해두고 제시한다. → 그래서 agent가 llm을 컨트롤해서 llm의 자율성이 낮은 방식
    2. fully autonomous (완전 자율) 
        1. ReAct구조 역할로, llm이 추론과 행동이 반복되는 과정
        2. llm이 어떤 추론을 할지 어떤 행동을 할지를 스스로 결정하는 것

# langgraph란

- 프로임워크로 대규모 언어모델인 llm을 활용한 복잡한 워크플로우와 의사결정 프로세스를 구현하도록 해준다.
    - 상하위 에이젼트를 순서대로 제어할 수 있다.
    - 하나의 에이젼트를 사용해도 되고, 멀티 에이젼트를 사용해도 된다.
- 멀티 Agent기반의 llm 어플리케이션을 효과적으로 만들 수 있는 강력한 기술
- langchain팀에서 직접 지원하는 프레임워크(호환성가짐), langchain외에도 호환성 가진다.
- StateGraph
    - 상태(state)를 기반으로 작동하는 그래프 구조로, 복잡한 워크플로우를 유연하고 제어가능하게 구축해준다.
    - 각 노드가 특정 상태를 나타내며, edge가 상태 간 전이 조건을 정의한다.
    - state(상태)
        - 어플리케이션의 현재 스냅샷을 나타내는 공유 데이터 구조로 주로 TypedDict나 Pydantic BaseModel을 사용한다.
    - Node
        - Agent로직을 인코딩하는 파이썬 함수
        - 현재 State를 입력으로 받고, 계산과 같은 것을 수행한 후, 업데이트된 Statue를 반환한다.
    - Edge
        - 현재 State를 기반으로 다음에 실행할 Node를 결정하는 함수로, 조건부 분기/고정 전이를 하며 시스템의 흐름을 제어한다.
- 조건부 Edge를 활용한 분기작업
    - 조건에 따라서 다음 노드를 선택적으로 흐르게 하기 위해
- State Reducer
    - 상태 업데이트를 관리하는 함수
    - 랭그래프는 상태를 공유한다. 각 노드들은 입력값을 받아서 상태를 업데이트해서 출력을 하게 되는데, 그러면서 다음 노드로 상태를 넘겨준다.
    - 동작방식
        - 기본: stategraph는 기존 값을 덮어쓰는 형태로 동작이 기본 리듀서의 동작방식이다.
        - add
        - custom : 중복제거, 정렬 등 사용자 정의 리듀서 지원
- MessageGraph
    - LangChain의 ChatModel을 위한 특수한 형태의 StateGraph
    - Message 리스트를 입력으로 처리해준다. → 대화가 자연스러워 지고, 컨텍스트 활용해서 답변이 가능하다.
    - 채팅 히스토리를 관리해주는 장점이있다.
    - 주의 : 오래된 메시지를 제거하는 메모리 관리 기능 추가 필요, 성능 최적화 필요, 민감한 정보의 메시지는 적절히 처리
- ReAct(Reasoning + Acting)
    - 추론과 행동을 결합한 접근 방식으로
    - llm이 단순히 텍스트를 생성하는 것을 넘어서, 추론과 행동의 반복적인 사이클을 통해 복잡한 작업을 단계적으로 해결
    - langraph에 내장된 ReAct에이젼트 사용
- MemorySaver
    - 랭그래프에는 상태가 일시적으로 동작한다는 문제가 있다.(각 실행마다 새로운 상태로 초기화시킴 → 대화가 연속적으로  안되는 문제가 된다.)
    - 그래서 그래프의 각 단계 실행 후 자동으로 상태를 저장한다(체크포인터 역할)
    - key-value로 저장시킨다.

# langchain vs langgraph

- langchain
    - 개발과정이 랭그래프에 비해 간단하지만, 복잡한 워크플로우 관리에는 한계가 있음
    - 즉, rag에서 검색해서 답변을 생성해내는 것과 같은 간단한 구조는 가능하지만, 이 답변에 대해 평가하고 답변이 부족했을때 검색을 한다거나 하는 워크플로우를 구현하는 과정은 복잡해지고, 상태관리가 한계가 있다.
    - 구조 : 체인 및 에이전트 기반
    - 용도 : 간단한 llm 어플리케이션, rag
    - 내장 tool
        - 검색 서비스 : DuckDuckGo, TavilySearch
- langgraph
    - 랭체인 기반이지만, 상태기반 그래프 구조를 통해 북접하고 동적인 워크플로우 구성
    - 구조: 그래프 기반
    - 용도 : 복잡한 AI시스템, 멀티 에이젼트

# LangChain ToolCalling

- llm이 외부 기능이나 데이터에 접근할수 있게 해주는 매커니즘으로,
- 기존에 llm의 한계(최신 정보 부족, 정확한 작업을 수행할 필요가 있거나, 코딩을 하거나 등)을 극복하는 방법
- 예를 들어서, 곱하기 연산을 llm이 직접수행하는 것이 아닌, 외부 도구를 통해서 정확성을 향상시키게 하는 것인데. 그 외부 도구가 API라고 한다면 인자값을 호출할 때 **구조화해서 표현**할 수 있게 = 즉, 자연어를 구조화있게 변환하는 것
- Tavily 웹 검색 도구 사용
    - [https://python.langchain.com/docs/integrations/tools/tavily_search/#tool-features](https://python.langchain.com/docs/integrations/tools/tavily_search/#tool-features)
    - tool calling 적용
        
        ```java
        from langchain_openai import ChatOpenAI
        
        # ChatOpenAI 모델 초기화
        llm = ChatOpenAI(model="gpt-4o-mini")
        
        query = "스테이크와 어울리는 와인을 추천해주세요."
        
        # Tavily 검색 도구 초기화 (최대 2개의 결과 반환)
        web_search = TavilySearchResults(max_results=2)
        
        # 웹 검색 도구를 직접 LLM에 바인딩하는 과정
        llm_with_tools = llm.bind_tools(tools=[web_search])
        
        # 쿼리를 llm에 전달 (invoke 호출)
        ai_msg = llm_with_tools.invoke(query) // llm이 검색에 맞는 적절한 검색어를 생성하게 된다. 
        ```
        
- 간단한 질의의 경우, 도구 호출이 필요없을 때는 llm이 자체적으로 툴을 적용하지 않기 때문에 too_calls를 프린트해보면 빈배열로 리턴된다.
- description도 llm에 전달되어 어떤 도구인지 파악하게 하기 떄문에 중요하다.
- Tool Calling 실행방법
    1. args 스키마 사용
        
        ```java
        tool_call = ai_msg.tool_calls[0]
        tool_output = web_search.invoke(tool_call["args"])
        ```
        
    2. tool_call 자체를 전달(일반적으로 사용되는 방식)
        
        ```java
        tool_message = web_search.invoke(tool_call)
        ```
        
    3. toolMessage를 정의해서 호출
        1. 메시지를 정교하게 호출할 경우에 사용
        
        ```java
        tool_message = ToolMessage(
        		content = tool_output,
        		tool_Call_id=tool_call["id"],
        		name=tool_call["name"]
        )
        ```
        
    4. 배치 호출
        1. 호출할 도구가 여러개인 경우, 한번에 호출할 필요가 있을 때
        
        ```java
        tool_messages = web_search.batch(ai_msg.tool_calls)
        ```
        
- 툴을 통해 얻은 결과를 다시 llm에게 전달하여 AI 답변 생성
    1. 프롬프트 템플릿 정의 - 시간 전달(실시간성이 중요하다면)
    2. LLM 모델 초기화
    3. LLM에 도구를 바인딩
    4. LLM 체인 생성
        
        ```java
        llm_chain = prompt | llm_with_tools
        ```
        
    5. llm체인에 toolMessage 전달
- 사용자 정의 도구
    - @tool 데코레이터를 사용해서 작성한다.
        - 함수를 langchain도구로 변환하는 방법으로
        - 명확한 입출력 정의해야한다.
        - @tool 데코레이터만 사용하면 랭체인이나 랭그래프에서 도구로 인식한다.
- Runnable 객체를 도구(tool) 변환
    - 문자열이나 dict 입력을 받는 Runnable을 도구로 변환해서 사용할 수 있다.
    - as_tool에서드를 사용한다.
- LCEL 체인을 도구로
    - 위키피디아 문서를 검색하고 내용을 요약하는 체인
- 백터 저장소 검색기
    - @tool 데코레이터를 사용한다
    - 실습 횐경
        - ollma bge-m3 db 설치(한국어 잘됨)
        - Croma db 사용
- Few-shop프롬프팅을 사용한 ToolCalling 성능개선
    - Few-shot 프롬프팅이란?
        - 모델에게 몇 가지 예시를 제공하여 원하는 출력 형식이나 작업 수행 방식을 보여주는 기법
        - 모델에게 도구를 어떻게 사용해야하는지 예시를 통해 보여주는 목적으로 사용
    - ToolCalling 적용과정
        1. 예시 생성
        2. 프롬프트 템플릿 생성
        3. 프롬프트 생성 및 모델 호출
        4. 응답 파싱
- LangChain Agent
    - 정의: LLM을 추론엔진으로 사용하여 어떤 행동을 할지, 그 행동의 입력은 무엇일지 결정하는 시스템
    - 어떤 행동을 할지, 어떤 입력값이 필요한지 결정하는 단계를 넘어서서 그 도구를 실행을 해서 검색이 필요하면 검색도구를 사용하고 이런 일련의 과정을 llm이 컨트롤하는 과정을 포함하는 것을 Agent라고 한다.
    - 언어 모델이 단순히 텍스트를 출력하는 것을 넘어 실제 행동을 취하게 한다.
    - 도구를 사용하고 사용된 행동의 결과를 다시 Agent에 피드백하여 추가 행동할지(즉 도구를 다시 호출할지) 또는 작업을 완료할지를 결정
    - create_tool_calling_agent : 도구호출을 담당하는 에이젼트를 정의하는 함수
    - AgentExecutor : 에이젼트를 실행시키는 함수로 상속받아서 실행시킨다.
- Gradio 활용
    - 머신러닝이나 인공지능 어플리케이션을 빠르게 웹 인터페이스로 구현해볼 수 있는 서비스이다.

# Adaptive Rag

- 질문의 복잡성에 따라 가장 적합한 검색 및 생성 전략을 동적으로 선택하는 방법
- 단순한 질문은 기본 llm으로 처리하고, 복잡한 질문은 반복적으로 rag를 검색하는 전략을 사용한다.

# 사람의 개입 HITL(Human-in-the-Loop)

- AI 시스템의 자동화된 처리와 인간의 전문 지식을 결합하는 것
- 시스템의 결정이나 출력에 대해 인간이 검토하고 개입할 수 있는 지점을 제공한다.

# Self-RAG

- 기존의 RAG모델에 자기 반영 능력을 추가한 확장모델로, 정보를 자기 평가(grade_generation)하고 통합해서 관련성 높은 응답을 하는 것이 목표
- 동적쿼리 적용

# 서브그래프 (Subgraphs)

- 하나의 노드에서 복잡하게 구성해야한다면, 개별 노드에서 일어나는 작업자체를 별도 노드의 독립적인 서브 그래프로 구현을 하고 전체 그래프 내에 통합시키는 방식으로 구현
- 복잡한 워크 플로우를 모듈화해서  구현이 가능하다.
- 병렬로 노드  실행을 시켜서 서브 그래프들은 각각 독립적으로  실행시켜야  한다. → 팬아웃과 팬인  매커니즘을  통통해  구현한다.

# Corrective RAG (CRAG) 개요

- 기존 RAG시스템을 개선하여 검색된 정보의 품질과 관련성을 향상시키는 접근 방식
- 문서 관련성 평가, 지식 정제, 필요시 외부 지식 탐색, 그리고 정제된 지식을 바탕으로 한 답변 생성
- 기존 RAG만으로는 충분한 답변이 안될때 - 지식 수정까지 하게 함

# 수강 후기

- 역시나 환경만들기가 제일 오래걸린다. 로컬 python으로 poetry가 설치가 한번에 안되었다. 오류의 오류..
- 중반 지나니까 개념만이라도 이해하고 넘어가는 것으로 목표를 바꾸게 되었다 😭
- Agent와의 개념이 잘 안잡혔는데, 그나마 좀 이 생태계가 2%는 알아간 느낌이다..
- 논문을 자주 찾아봐야겠다는 생각이 들었다.
    - [https://arxiv.org/search/cs?query=crag&searchtype=all&abstracts=show&order=-announced_date_first&size=50](https://arxiv.org/search/cs?query=crag&searchtype=all&abstracts=show&order=-announced_date_first&size=50){:target="_blank"}