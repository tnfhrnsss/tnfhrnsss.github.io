---
layout: post
title: 생성형 AI 입문자를 위한 LangChain 기초
date: 2024-08-16 15:04:00
last_modified_at : 2024-08-16 15:04:00
parent: Inflearn
grand_parent: Mooc
nav_exclude: true
---


# 강의 정보

강의 URL : [https://inf.run/f5aQg](https://inf.run/f5aQg)  
강사: 오승환님  
총 수강시간: 1시간 3분  
실습파일 : https://github.com/tsdata/langchain-study  
실습환경: [코랩](https://colab.google/){:target="_blank"}   
비용 : 강의는 무료이지만, 테스트가 되려면, gpt에 billing을 추가해야 해당 모델을 사용할 수 있다.  

# 학습 목표

- 랭체인 기본 개념과 사용법 실습
- rag 기법 이해

# 선수학습

- 파이썬
- 약간의 머신러닝

강의 내용 기반으로 정리했습니다.
궁금한 것들 추가적으로 찾아봐서 정리했습니다.

# Q. 기존에도 gpt api에 프롬프트 요청해서 응답을 얻는 것은 가능했던것인데 굳이 랭체인을 사용하는 이유는 무엇인가? 어떤 장점이 있는가?

- 내 개인 생각으론 멀티 체인과 결과 파싱인 것 같다. 아무래도 결과에서 데이터 뽑아오는 구조가 일반적?이지 못한 부분인데 result.content로 텍스트만 얻을 수 있어서 이런 부분이 좋다.

## 1. **복잡한 워크플로우 관리**

- 단순히 GPT API에 프롬프트를 보내는 것과 달리, LangChain은 다양한 작업을 순차적으로 또는 병렬로 처리할 수 있는 기능이 있다.
    - **체인(Chains)**: 예를 들어, 데이터 수집, 전처리, 모델 학습 및 예측과 같은 복잡한 작업을 하나의 체인으로 구성해서 여러 단계의 처리를 하나로 관리할 수 있게 한다.
    - **에이전트(Agents)**: LangChain은 다양한 에이전트를 제공하여 작업의 흐름을 동적으로 제어할 수 있다.

## 2. **유연한 프롬프트 관리**

- **프로프트 템플릿(Prompt Templates)**: 다양한 상황에 맞게 프롬프트를 동적으로 생성할 수 있다.
- **프롬프트 체인(Prompt Chaining)**: 여러 프롬프트를 조합하여 복잡한 질문이나 작업을 처리할 수 있다.

## 3. **다양한 데이터 소스 및 도구와의 통합**

- **데이터 커넥터(Data Connectors)**: 다양한 데이터베이스, API, 파일 시스템 등과 통합하여 실시간 데이터를 가져오고 활용할 수 있다.
- **툴(도구) 사용**: 예를 들어, 수학적 계산, 정보 검색, 데이터베이스 쿼리 등 다양한 툴을 사용하여 모델의 기능을 확장할 수 있다.

## 4. **상태 관리 및 메모리 기능**

- **메모리(Memory)**: 대화 상태와 문맥을 관리할 수 있는 이전 대화 내용을 기억하고 재사용할 수 있는 메모리 기능이 있다.
- **상태 관리(State Management)**: 작업의 진행 상황과 결과를 추적하고 관리할 수 있다.

## 5. **사용자 정의 및 확장성**

- **사용자 정의 체인 및 에이전트**: 사용자가 자신의 필요에 따라 체인과 에이전트를 정의하고 사용할 수 있다.
- **확장 가능한 아키텍처**: LangChain은 다양한 기능을 추가하고 확장할 수 있는 유연한 아키텍처를 제공한다.

---

# 강의1. 체인에 대한 이해

- 기본 체인 = Prompt + LLM
    - 3단계 구조 : Prompt Format → LLM or Chat Model Predict → Parse
- 프롬프트를 받아 llm을 통해 응답을 생성하는 구조
- 셋팅 구성
    - langchain
        
        ```python
        pip install -q langchain langchain-openai tiktoken
        ```
        
- 프롬프트 템플릿 구성 및 기본체인
    
    ```python
    from langchain_openai import ChatOpenAI
    from langchain_core.prompts import ChatPromptTemplate
    from langchain_core.output_parsers import StrOutputParser
    
    # prompt + model + ouput parser
    prompt = ChatPromptTemplate.from_template("You car an export in astronomy. Answer the question. <Question>: {input}")
    llm = ChatOpenAI(model = "gpt-3.5-turbo-0125")
    output_parser = StrOutputParser()
    
    # LCEL chaining
    chain = prompt | llm | output_parser
    
    # chain 호춯
    chain.invoke({"input": "지구의 자전 주기는?"})
    ```
    

- 멀티 체인
    - 2개 이상의 체인을 순차적으로 연결 수행
    
    ```python
    prompt1 = ChatPromptTemplate.from_template("translates {korean_word} to English.")
    prompt2 = ChatPromptTemplate.from_template("explain {english_word} using oxford dictionary to me in Korean.")
    
    llm = ChatOpenAI(model = "gpt-3.5-turbo-0125")
    chain1 = prompt1 | llm | StrOutputParser()
    
    chain1.invoke({"korean_word": "미래"})
    
    chain2 = (
        {"english_word": chain1}
        | prompt2
        | llm
        | StrOutputParser()
    )
    
    chain2.invoke({"korean_word": "미래"})
    ```
    

# 강의 2. 프롬프트 만들기(PromptTemplate)

- 핵심은 재사용 가능한 프롬프트 구성
- 환경 구성
    
    ```python
    !pip install -q langchain langchain-openai tiktoken
    ```
    

### 1. PromptTemplate

- 단일 문자 입력하면 단일문장 출력하는 체인을 사용할때
- langchaing의 prompttemplate은 문자열 기반이기때문에 문자열 템플릿끼리 결합해서 사용할 수 있다.
    
    ```python
    from langchain_core.prompts import PromptTemplate
    
    # name과 age라는 두개의 변수를 사용하는 프롬프트 템플릿을 정의
    template_text = "안녕하세요. 제이름은 {name}이고, 나이는 {age}살입니다."
    
    # PromptTemplate 인스턴스를 생성
    prompt_template = PromptTemplate.from_template(template_text)
    
    # 템플릿에 값을 채워서 프롬프트를 완성
    filled_prompt = prompt_template.format(name="홍길동", age=30)
    filled_prompt
    
    combined_prompt = (
        prompt_template
        + PromptTemplate.from_template("아버지를 아버지라 부를 수 없습니다.")
        + " {language}로 번역해주세요."
    )
    
    combined_prompt
    ```
    

## 2. 챗 프롬프트 템플릿(ChatPromptTemplate)

- 대화형 상황에서 여러 메시지 입력을 기반으로 단일 메시지로 된 응답을 생성할 때 사용
- 대화형 모델이나 챗봇 개발에 주로 사용
- 구성 : 각 채팅 메시지는 역할과 내용이 짝을 이루는 형태
- ChatPromptTemplates + ChatModels
- 메시지 구성방법
    - 1안) 튜플 활용
        
        ```python
        from langchain_core.prompts import ChatPromptTemplate
        
        chat_prompt = ChatPromptTemplate.from_m
        	essages(
            [
                ("system", "이 시스템은 천문학 질문에 답변할 수 있습니다."),
                ("user", "{user_input}"),
            ]
        )
        
        messages = chat_prompt.format_messages(user_input="태양계에서 가장 큰 행성은 무엇인가요?")
        messages
        
        chain = chat_prompt | llm | StrOutputParser()
        
        chain.invoke({"user_input": "태양계에서 가장 큰 행성은 무엇인가요?"})
        ```
        
    - 2안) MessagePromptTemplate 활용
        - 역할에 따른 SystemMessagePromptTempate, HumanMessagePromptTemplate, AiMessagePromptTemplate 등등이 있다.
        
        ```python
        from langchain_core.prompts import SystemMessagePromptTemplate, HumanMessagePromptTemplate
        
        chat_prompt = ChatPromptTemplate.from_messages(
            [
                SystemMessagePromptTemplate.from_template("이 시스템은 천문학 질문에 답변할 수 있습니다."),
                HumanMessagePromptTemplate.from_template("{user_input}")
            ]
        )
        
        messages = chat_prompt.format_messages(user_input="태양계에서 가장 큰 행성은 무엇인가요?")
        messages
        ```
        

# 강의3. llm 모델 구조

- 랭체인 모델 유형
    - llm, chat model 클래스 두개의 모델이 있다.

## 1. llm

- llm은 단일 질문에 대한 복잡한 출력을 생성
- 광범위한 언어 이해 및 텍스트 생성 작업에 사용

```python
from langchain_openai import OpenAI

llm = O

penAI()

llm.invoke("한국의 대표적인 관광지 3군데를 추천해주세요.")
```

## 2. chat model

- 연속적으로 대화하는 형태의 상호작용
- 여러개의 메시지를 입력으로 받아서 하나의 메시지를 반환하는 대화형 상황에 최적화

```python

from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

chat = ChatOpenAI()

chat_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "이 시스템은 여행 전문가입니다."),
        ("user", "{user_input}"),
    ]
)

chain = chat_prompt | chat
chain.invoke({"user_input" : "안녕하세요? 한국의 대표적인 관광지 3군데 추천해주세요."})

```

# 강의4. llm 모델 튜닝

- 모델 파라미터 설정
    - temperature : 값이 크면 다양하고 예측이 어려운 출력이 생성.
- 설정 시점
    - 방법1) 모델 인스턴스 생성할때 설정값을 인자로 전달하는 방법
    - 방법2) 모델에 직접 파라미터를 전달(모델 호출 시점)
        
        ```python
        model.invoke(input=question, **params)
        ```
        
    - 방법3) 모델에 추가적인 파라미터 전달
        - bind메서드를 통해서 새로운 파라미터 전달
            
            ```python
            chain = prompt | model.bind(max_tokens=10)
            ```
            

# 강의5. RAG 기법 이해

- rag란?
    - llm을 확장해서, 더욱 정확하고 풍부한 정보를 제공하는 기법으로 모델에 포함되지 않은 외부데이터를 실시간으로 검색하고 리를 기반으로 답변 생성하는 과정으로, 환각을 방지할 수 있다.
    - 실시간 정보가 반영되는 장점이 있다.
    - rag에선 환각 방지 위해 temperatur를 낮춘다.
- 기본 구조
    - 검색단계 → 생성단계
    - 파이프라인
        - load data → text split → indexing → retrieval → generation

```python
!pip instal

l -q la

ngchain langchain-openai tiktoken chromadb

## 
import langchain

langchain.__version__

## 1. Data load
from langchain_community.document_loaders import WebBaseLoader

# 위키피디아 정책과 지침
url = 'http://...'
loader = WebBaseLoader(url)

# 웹페이지 텍스트 -> Documents
docs = loader.load()

## Text split

from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = text_splitter.split_documents(docs)

## Indexing (texts -> embedding -> store)

from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings

vectorstore = Chroma.from_document(documents = splits, embedding=OpenAIEmbeddings())

docs = vectorstore.similarity_search("격하 과정에 대해서 설명해주세요)")

## Retrieval - Generation
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser

# Prompt
template = ''' Answr the question based only on the following context:
[context]

Question: {question}
'''

prompt = ChatPromptTemplate.from_template(template)

# LLM
model = ChatOpenAI(model='gpt-3.5-turbo-0125', temperature=0)

# Retriever
retriever = vectorstore.as_retriever()

# Combine Documents
def format_docs(docs):
  return '\n\n'.join(doc.page_content for doc in docs)

# Rag Chain 연결
rag_chain = (
    {'context': retriever | format_docs, 'question': RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# Chain 실행
rag_chain.invoke("격하 과정에 대해서 설명해주세요.")
```