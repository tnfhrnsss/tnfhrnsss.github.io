---
layout: post
title: DocentAI(도센타이) Gemini 3 해커톤 참여 후기
description: Gemini 3 해커톤에서 Netflix 자막 맥락 설명 크롬 익스텐션 DocentAI를 만든 과정
date: 2026-05-11 10:00:00
last_modified_at : 2026-05-11 10:00:00
parent: Sub Projects
has_children: false
nav_exclude: true
---

# Gemini 3 해커톤

- Gemini 3 해커톤은 Google이 주최한 [Gemini API Developer Competition 3](https://gemini3.devpost.com/){:target="_blank"}로
- Gemini API를 활용한 앱이나 서비스를 만드는 대회로
- 작년 12월 말부터 참여해서 제출까지 완료했습니다.

---

# DocentAI 프로젝트

## 1. 기획

### 배경

- 넷플릭스 드라마나 영화를 볼 때 가끔 "저 사람 누구였지?", "이 장면이 왜 나오지?" 싶은 순간이 생깁니다.
- 그럴 때마다 멈추고 검색을 해봐야했고, 그럴 때마다 흐름이 끊기는 문제가 있었습니다.
- 이 불편함을 AI로 해결해보자는 아이디어에서 출발했습니다.

### 컨셉

- 박물관 도슨트(docent)처럼, 작품을 해설해주는 AI 가이드
- 언어 학습 도구가 아닌 **맥락 이해 도구**
- 시청을 멈추지 않고 `Ctrl+E` 한 번으로 맥락 설명을 받는 경험

### 유저 스토리

- 넷플릭스에서 영상을 보다가 헷갈리는 자막이나 장면이 나왔을 때 `Ctrl+E`를 누른다.
- 현재 자막, 에피소드 정보, 이전 대화 맥락을 기반으로 AI가 설명해준다.
- 화면 캡처도 가능해서 시각적 맥락까지 함께 분석할 수 있다.
- 한국어/영어 다국어로 설명을 받을 수 있다.

---

## 2. 아키텍처

두 개의 레포지토리로 구성했습니다.

```
┌──────────────────┐       ┌──────────────────┐       ┌─────────────────────┐
│ Chrome Extension │ ←───→ │  FastAPI Backend  │ ←───→ │  Google Gemini AI   │
│  (docentai-ui)   │ HTTPS │  (GCP Cloud Run)  │  AI   │  + Search Grounding │
└──────────────────┘       └──────────────────┘       └─────────────────────┘
```

### 2-Step 구조

비용 최적화를 위해 2단계 아키텍처를 설계했습니다.

```
STEP 1: 영상 등록
  └─> Gemini Search Grounding으로 작품 정보 수집 (최초 1회)
  └─> DB에 레퍼런스 저장

STEP 2: 설명 요청 (여러 번)
  └─> 저장된 레퍼런스 활용
  └─> 빠르고 비용 효율적
```

### 멀티모달 분석

텍스트만이 아닌 다양한 컨텍스트를 조합합니다.

- **텍스트**: 자막 + 이전 대화 히스토리
- **이미지**: 화면 캡처 (개발 빌드)
- **비언어 큐**: `[Sound effects]`, `(Facial expressions)` 등 자막 내 묘사 정보

---

## 3. 기술 스택

| 구분 | 기술 |
|------|------|
| Frontend | Chrome Extension, Vanilla JS |
| Backend | Python, FastAPI |
| AI | Google Gemini 3 Flash, Search Grounding |
| DB | SQLite |
| Infra | GCP Cloud Run |
| Auth | JWT |

---

## 4. 개발 중 문제와 해결

### 정확도 문제

- 처음엔 현재 자막만 프롬프트에 넣었더니 답변이 너무 얕았습니다.
- Google Custom Search API를 별도로 연동해봤지만, 동명이인/동명 작품 구분 등 정확도가 낮았습니다.
- Gemini의 **Search Grounding** 기능으로 전환하면서 정확도가 올라가게 되었습니다.
- 자막 중 `[소리 묘사]`, `(행동 묘사)` 등 비언어 정보를 더해서 맥락 품질을 높여보려고 했습니다..
---

## 5. 링크

- [Devpost 제출 페이지](https://devpost.com/software/docentai-your-ai-docent-for-netflix){:target="_blank"}
- [Frontend GitHub (docentai-ui)](https://github.com/tnfhrnsss/docentai){:target="_blank"}
- [Backend GitHub (docentai-core)](https://github.com/tnfhrnsss/docentai-core){:target="_blank"}
- [데모 영상](https://youtu.be/BUbfO1P8-Bs){:target="_blank"}

---

# 후기

- 셀프 환상에 빠졌었나 싶을 정도로 너무 재밌었습니다.
- 1등 수상작품을 보고나니, 각성되어 아마 다음 4회도 참여할 것 같습니다.ㅋㅋ
- 과정 중 산출물 준비가 제일 힘들었던 것 같습니다. 이것도 클로드의 도움을 받아서 진행했지만, 특히 영상제작은 노하우가 없다보니 확신이 안서서 오래걸린 것 같습니다.
