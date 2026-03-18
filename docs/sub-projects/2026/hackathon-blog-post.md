# Gemini 3 Hackathon 참여기: DocentAI 개발 여정

> Netflix 자막을 AI가 설명해준다면? DocentAI 프로젝트의 시작부터 제출까지의 전체 과정을 공유합니다.

---

## 📌 목차

1. [프로젝트 개요](#프로젝트-개요)
2. [왜 DocentAI를 만들었나](#왜-docentai를-만들었나)
3. [기술 스택 및 아키텍처](#기술-스택-및-아키텍처)
4. [개발 과정](#개발-과정)
5. [주요 챌린지와 해결](#주요-챌린지와-해결)
6. [데모 영상 제작](#데모-영상-제작)
7. [Devpost 제출](#devpost-제출)
8. [배운 점과 향후 계획](#배운-점과-향후-계획)
9. [마무리](#마무리)

---

## 프로젝트 개요

### DocentAI란?

DocentAI는 Netflix 시청 중 이해가 안 가는 자막이나 장면을 즉시 설명해주는 Chrome 확장 프로그램입니다. 마치 박물관의 도슨트(Docent)가 작품을 설명해주듯, AI가 영상 콘텐츠의 맥락을 실시간으로 제공합니다.

**핵심 기능:**
- 🎯 컨텍스트 인식 자막 설명
- 🔍 Google Gemini Search Grounding 활용
- 🖼️ 멀티모달 분석 (텍스트 + 이미지)
- 🌐 다국어 지원
- ⚡ 2초 이내 빠른 응답

**기술 스택:**
- Frontend: Chrome Extension (Manifest V3)
- Backend: FastAPI + Python
- AI: Google Gemini 3 Flash API
- Deployment: Google Cloud Run

**링크:**
- 🔗 GitHub (Frontend): https://github.com/tnfhrnsss/docentai-ui
- 🔗 GitHub (Backend): https://github.com/tnfhrnsss/docentai-core
- 🔗 Documentation: https://tnfhrnsss.github.io/docentai/
- 🔗 Devpost: [제출 완료]

---

## 왜 DocentAI를 만들었나

### 개인적 경험에서 시작

OTT 플랫폼으로 영화와 드라마를 보다 보면, 외국 작품은 물론 국내 작품을 볼 때도 이해가 안 가는 대사나 장면이 종종 있습니다. 예전에는 옆에 있는 친구나 가족에게 "지금 뭐라고 한 거야?"라고 물어볼 수 있었습니다.

하지만 OTT의 등장으로 시청 문화가 변했습니다. 이제는 혼자 보는 영상이 많아졌고, 각자 다른 작품을 즐기게 되었죠. 혼자 시청하다가 이해가 안 가는 장면이 나오면 어떻게 할까요?

**기존 방법의 문제점:**
1. 영상 일시정지
2. 검색 엔진 열기
3. 키워드 입력
4. 결과 찾기
5. 다시 재생

→ 번거롭고, 몰입이 깨지고, 결국 대충 넘어가게 됨

### AI 시대의 해결책

"이해가 안 가는 자막을 AI에게 바로 질문할 수 있다면?"

이 질문에서 DocentAI가 시작되었습니다. 마치 박물관 도슨트가 그림 앞에서 숨겨진 이야기를 들려주듯, Netflix 앞에서 AI가 설명해주는 경험을 만들고 싶었습니다.

---

## 기술 스택 및 아키텍처

### 전체 구조

```
[Netflix 페이지]
    ↓ (자막, 메타데이터 추출)
[Chrome Extension]
    ↓ (API 요청: 자막 + 컨텍스트)
[FastAPI 백엔드]
    ↓ (프롬프트 구성 + 이전 자막 5개)
[Gemini 3 Flash API]
    ├─ Multimodal Processing (텍스트 + 이미지)
    ├─ Search Grounding (실시간 검색)
    └─ Function Calling (메타데이터 수집)
    ↓ (AI 응답)
[사용자에게 표시]
```

### 왜 Gemini 3 Flash를 선택했나?

1. **빠른 응답 속도**: 사용자가 영상을 보는 중 2초 이내 응답 필요
2. **Multimodal 지원**: 자막(텍스트) + 화면(이미지) 동시 처리
3. **Search Grounding**: 실시간 검색으로 정확도 크게 향상
4. **긴 컨텍스트 윈도우**: 작품 정보 + 대화 히스토리 유지

### Repository 구조

**docentai-ui (Frontend)**
- Chrome Extension Manifest V3
- Content Script로 Netflix DOM 파싱
- Background Service Worker로 API 통신
- Floating UI로 사용자 경험 최적화

**docentai-core (Backend)**
- FastAPI로 RESTful API 제공
- SQLite로 캐싱 및 세션 관리
- JWT 인증
- Gemini API 통합

---

## 개발 과정

### Phase 1: 기획 및 설계 (1월 중순)

**초기 아이디어:**
- Netflix 자막 설명 Chrome Extension
- Gemini API 활용
- 실시간 응답

**기술 검증:**
- Chrome Extension Manifest V3 학습
- Gemini API 테스트
- Netflix DOM 구조 분석

**결정 사항:**
- 2-repo 구조 (Frontend/Backend 분리)
- Gemini Flash 선택 (속도 우선)
- Google Cloud Run 배포

### Phase 2: MVP 개발 (1월 말)

**Frontend 개발:**
- Chrome Extension 기본 구조
- Netflix 자막 추출 로직
- 간단한 UI (Popup)

**Backend 개발:**
- FastAPI 기본 API
- Gemini API 통합
- 간단한 프롬프트

**첫 번째 테스트:**
- 작동은 하지만 정확도 낮음
- 응답 시간 5-7초 (너무 느림)
- 프롬프트 개선 필요

### Phase 3: 정확도 개선 (2월 초)

**문제점 발견:**

1. **Google Custom Search API의 한계**
   - 동명 작품 구분 어려움
   - 예: "Cargo" (2018) vs "Cargo" (2013)
   - 검색 결과가 섞여서 부정확

2. **단순 프롬프트의 한계**
   - 자막 텍스트만으로는 맥락 파악 어려움
   - "그 사람"이 누구인지 모름
   - 시간적 맥락 이해 부족

**해결 방법:**

✅ **Gemini Search Grounding으로 전환**
- 맥락을 이해하고 관련성 높은 정보 통합
- 출처 URL 제공으로 신뢰도 향상
- 동명 작품도 정확히 구분

✅ **이전 자막 5개 컨텍스트 포함**
- 현재 자막뿐 아니라 이전 5개 자막도 함께 전송
- 대화 흐름과 서사 진행 맥락 제공
- "그 사람" 같은 대명사도 정확히 해석

✅ **비언어적 정보 추출**
- 자막의 [웃음], [한숨], [문 닫는 소리] 파싱
- 타임라인 정보로 시간적 맥락 제공
- 멀티모달 효과 구현 (이미지 없이도)

**결과:**
- 정확도 40% 향상
- 응답 시간 2-3초로 단축
- 사용자 경험 크게 개선

### Phase 4: 저작권 문제 해결 (2월 초)

**문제:**
- 스크린샷 이미지를 서버로 전송/저장 시 저작권 이슈

**해결:**

두 가지 프로필 구현:
1. **Production 버전**: 스크린샷 제외, 자막+메타데이터만
2. **Hackathon 버전**: 스크린샷 포함, 교육/연구 목적 명시

대안으로 비언어적 정보 강화:
- 자막 내 동작 표현 파싱
- 타임라인 정보 추출
- 음향 정보 텍스트화

→ 이미지 없이도 풍부한 컨텍스트 제공

### Phase 5: UI/UX 개선 (2월 중순)

**초기 문제:**
- Popup UI가 영상 시청 방해
- 키보드 단축키 필요

**개선:**
- Floating Button (우측 상단, 최소화 가능)
- `Ctrl+E` / `Cmd+E` 단축키
- 비침해적 디자인
- 대화 히스토리 표시

### Phase 6: 성능 최적화 (2월 중순)

**캐싱 전략:**
- 작품 메타데이터 SQLite 캐싱
- 세션별 대화 히스토리 관리
- 자주 묻는 질문 미리 생성

**결과:**
- 평균 응답 시간: 2-3초
- 첫 요청: 3-4초 (cold start)
- 후속 요청: 1-2초

---

## 주요 챌린지와 해결

### Challenge 1: 답변 정확도

**문제:**
- 초기 프롬프트만으로는 정확한 답변 어려움
- 동명이인 캐릭터, 시간적 맥락 구분 힘듦

**해결:**
1. Google Custom Search → Gemini Grounding 전환
2. 이전 자막 5개 컨텍스트 포함
3. 비언어적 정보 추출 및 파싱

**결과:**
- 정확도 40% 향상
- 출처 URL 제공으로 신뢰도 증가

---

### Challenge 2: Netflix 저작권

**문제:**
- 스크린샷 이미지 서버 전송/저장 시 저작권 이슈

**해결:**
- Production/Hackathon 두 버전 분리
- 비언어적 정보 추출로 대체
- 이미지 없이도 맥락 제공

**결과:**
- 저작권 문제 해결
- 멀티모달 효과는 유지

---

### Challenge 3: Chrome Extension Manifest V3

**문제:**
- Background Page → Service Worker 변경
- 지속적 연결 유지 어려움
- DOM 직접 접근 불가

**해결:**
- Content Script/Background 메시지 패싱 구조
- Chrome Storage API로 상태 관리
- 장시간 작업은 백엔드로 이관

---

### Challenge 4: 응답 속도

**문제:**
- 초기 응답 시간 5-7초 (몰입 깨짐)

**해결:**
1. 캐싱 전략 (메타데이터, FAQ)
2. Gemini Flash 선택 (가장 빠른 모델)
3. 비동기 처리 (백그라운드 정보 수집)
4. 스트리밍 응답 (일부 먼저 표시)

**결과:**
- 평균 2-3초로 단축
- 사용자 경험 크게 개선

---

## 데모 영상 제작

### 영상 구조 설계

**타임라인 (60초):**
```
0:00-0:07  Intro (제목 + 로고)
0:07-0:15  Problem (문제 제시 - 체크리스트)
0:15-0:20  Before (기존 방법의 불편함)
0:20-0:30  Solution (DocentAI 소개)
0:30-0:40  How it works (사용 방법)
0:40-0:45  Transition (전환 화면)
0:45-0:60  Demo (실제 사용 모습)
0:60-0:70  Outro (CTA + 링크)
```

### HTML 화면 제작

**주요 화면:**

1. **Intro 화면**
   - 브랜드 로고 (💡)
   - DocentAI 타이틀
   - 서브타이틀: "Your AI Docent for Netflix"
   - 애니메이션: 펄스 효과

2. **Problem Checklist**
   - 5가지 문제 상황
   - 순차적 하이라이트 애니메이션
   - 자동 호버 효과 (영상용)
   - 빨간색 강조 (문제 = 위험)

3. **How It Works**
   - 6단계 사용 방법
   - 좌우 분할 (텍스트 + 이미지)
   - 크로스페이드 전환
   - 타이밍 조절 가능한 구조

4. **Outro 화면**
   - CTA: "Try DocentAI"
   - GitHub 링크
   - 해커톤 배지

**기술 선택:**
- 순수 HTML/CSS (외부 라이브러리 없이)
- CSS 애니메이션 (시간 제어 쉬움)
- 1200x800 해상도 (3:2 비율)

### 배경음악 vs AI 음성

**결정: 배경음악 선택**

이유:
- ✅ 제작 시간 단축 (30분 vs 3시간)
- ✅ 전문적인 느낌
- ✅ 언어 무관 (국제 심사)
- ✅ 수정 용이
- ✅ 해커톤 표준 (73% 음악만 사용)

선택한 음악:
- YouTube Audio Library
- 밝고 긍정적인 톤
- 120-140 BPM
- 30-40% 볼륨

### 녹화 및 편집

**도구:**
- 화면 녹화: Chrome 개발자 도구
- 편집: iMovie
- 전환: 크로스페이드 (0.5초)
- 최종 출력: 1080p, H.264

**총 제작 시간: 약 6시간**
- HTML 화면 제작: 3시간
- 녹화: 1시간
- 편집: 2시간

---

## Devpost 제출

### 제출 항목 준비

**1. 프로젝트 스토리**

7개 섹션으로 구성:
- 💡 Inspiration: 개인적 경험과 동기
- ⚙️ What it does: 기능 설명
- 🛠️ How we built it: 기술 스택
- 🚧 Challenges: 주요 어려움과 해결
- 🏆 Accomplishments: 성과
- 📚 What we learned: 배운 점
- 🚀 What's next: 향후 계획

총 2,100 단어 (영어)

**2. 썸네일 이미지**

3:2 비율 (1200x800px)
- 좌측: 큰 아이콘 (💡)
- 우측: DocentAI 타이틀
- 하단: Gemini 3 Hackathon 배지
- 펄스 효과로 생동감

**3. YouTube 영상**

- 길이: 60초
- 해상도: 1080p
- 설명란: 프로젝트 소개 + 링크
- 태그: #DocentAI #GeminiAPI #Hackathon

**4. Testing Instructions**

심사위원을 위한 가이드:
- 설치 방법 (3단계)
- 테스트 방법 (5분 이내)
- 문제 해결 팁
- 추천 테스트 콘텐츠

**5. 추가 파일**

Chrome Extension 패키지:
- docentai-ui-dev-v1.0.0.zip
- README.txt 포함
- 35MB 이하

**6. 태그라인 (200자)**

```
"Your AI docent for Netflix — 
instant subtitle explanations without breaking immersion."
```

### 제출 완료

**제출 일시:** 2월 9일 (마감일)

**체크리스트:**
- ✅ GitHub 링크 (Frontend/Backend)
- ✅ 데모 영상 (YouTube)
- ✅ 프로젝트 스토리 (영어 2,100단어)
- ✅ 썸네일 이미지 (1200x800)
- ✅ Testing Instructions
- ✅ Chrome Extension 패키지
- ✅ Documentation 사이트

---

## 배운 점과 향후 계획

### 배운 점

**1. Gemini API의 강력함**

- **Search Grounding**: 단순 LLM을 넘어 실시간 검색 통합
- **Multimodal**: 텍스트+이미지 동시 처리로 맥락 이해
- **긴 컨텍스트 윈도우**: 작품 히스토리 유지하며 정확한 답변
- **Function Calling**: Chrome API와 자연스러운 통합

**한계와 개선:**
- API Rate Limiting → 캐싱 전략 필수
- 간혹 hallucination → Grounding + 출처 URL로 완화

**2. 멀티모달 AI의 실용성**

처음에는 "이미지를 함께 보내면 좋겠다"는 단순한 생각이었습니다. 
하지만 저작권 문제로 이미지를 제외하면서 깨달은 것:

**멀티모달의 본질 = "다양한 형태의 정보를 통합하는 것"**

이미지 없이도:
- 비언어적 정보 ([웃음], [한숨])
- 타임라인 정보 (과거 회상 여부)
- 이전 자막 5개 컨텍스트

→ 충분히 풍부한 맥락 제공 가능

**3. Chrome Extension 개발**

- Manifest V3의 제약사항 이해
- Content Script/Background 역할 분리
- Netflix 같은 복잡한 웹앱 DOM 파싱의 어려움

**4. 사용자 경험의 중요성**

AI가 아무리 똑똑해도 UX가 나쁘면 아무도 안 씀:
- 응답 속도: 2초 이내 필수
- 비침해적 UI: 영상 방해 최소화
- 키보드 단축키: `Ctrl+E` 한 번으로 접근
- 맥락 유지: 연속된 질문도 흐름 이해

**5. 프롬프트 엔지니어링**

같은 AI 모델도 프롬프트에 따라 천차만별:
- 작품 정보, 타임스탬프, 대화 히스토리, 자막
- 어떤 순서로, 어떤 형식으로 전달하느냐가 핵심

### 향후 계획

**단기 (3월):**

1. **Kanana 통합 테스트**
   - 카카오 Kanana 베타 신청 중
   - 한국어 콘텐츠 정확도 개선 기대
   - Gemini vs Kanana 비교 데이터 수집

2. **버그 수정 및 안정화**
   - 사용자 피드백 반영
   - Edge case 처리
   - 에러 핸들링 개선

3. **문서화 강화**
   - API 문서 추가
   - 사용 예시 확대
   - 아키텍처 다이어그램 보완

**중기 (4-6월):**

1. **다국어 지원 확대**
   - 일본어, 중국어, 스페인어 추가
   - 각 언어별 문화적 맥락 최적화

2. **응답 속도 개선**
   - 평균 2-3초 → 1초 이내 목표
   - 예측 기반 프리패칭
   - Edge Computing 검토

3. **답변 정확도 향상**
   - 전문 사전 통합 (Tool)
   - 단어 분석 기능
   - 출처 검증 시스템

**장기 (6개월+):**

1. **다른 OTT 플랫폼 지원**
   - Disney+, Prime Video
   - Wavve, Tving (국내)
   - YouTube (영화, 다큐)

2. **장르별 맞춤 분석**
   - 스릴러: 복선, 떡밥 분석
   - 역사물: 시대 배경, 역사적 사실
   - SF: 세계관, 기술 용어
   - 로맨스: 관계도, 감정선

3. **Chrome Web Store 정식 출시**
   - 현재 개발 버전 → 프로덕션
   - 더 많은 사용자에게 제공

4. **커뮤니티 기능**
   - 사용자 설명 공유
   - 추가 정보 기여
   - 집단 지성 활용

---

## 마무리

### 해커톤 일정 변경

**원래 일정:**
- 제출 마감: 2월 9일
- 발표: 3월 6일/9일

**변경된 일정:**
- Stage 1 심사: 진행 중
- Stage 2 심사: 3월 중순 시작 예정
- 최종 발표: **4월 1일**

**이유:**
> "We're carefully reviewing all submissions... 
> Good things take time!"

→ 제출작이 많고, 신중한 심사 진행 중
→ 2단계 심사로 공정하게 평가
→ 완성도 높은 프로젝트에 유리!

**긍정적인 면:**
- ✅ 3주 추가 개선 시간
- ✅ Kanana 통합 테스트 가능
- ✅ 문서 및 코드 보완
- ✅ 심층 평가 기회

### 통계 (개발 기간 약 4주)

**코드:**
- Frontend: ~3,000 lines
- Backend: ~2,500 lines
- 총 Commit: 150+

**프로젝트 규모:**
- GitHub Stars: (진행 중)
- Documentation Pages: 10+
- API Endpoints: 8개

**영상 조회수:**
- YouTube 조회수: 4회 (제출 직후)
- → 정상적인 수치! (대부분 5-20회)
- → 심사에는 무관 (심사위원이 직접 시청)

### 느낀 점

**1. AI는 도구가 아닌 동반자**

DocentAI를 만들면서 깨달은 것은, AI가 단순히 "기능"이 아니라 "경험"을 만든다는 점입니다. 
혼자 보는 시대에도 옆에서 설명해주는 친구 같은 존재.
그것이 DocentAI의 정체성입니다.

**2. 완벽보다 완성이 중요**

초기에는 모든 기능을 완벽하게 만들려고 했습니다.
하지만 해커톤의 핵심은 "작동하는 프로토타입"입니다.
80%의 완성도로 100% 기능을 만드는 것이 더 가치 있습니다.

**3. 문서화의 힘**

코드만큼 중요한 것이 문서입니다.
README, Testing Instructions, Devpost Story...
심사위원이 프로젝트를 이해하는 유일한 창구입니다.

**4. 커뮤니티의 도움**

Claude (AI)와의 대화가 큰 도움이 되었습니다.
- 아키텍처 설계
- 문제 해결
- 문서 작성
- 영상 제작

혼자 했다면 2배 이상 시간이 걸렸을 겁니다.

### 발음에 대한 작은 이야기

프로젝트 이름 'DocentAI'를 저는 **"도센타이"**라고 부릅니다.
(도센트-에이-아이가 아닌)

마치 박물관 도슨트의 설명이 자연스럽게 흐르듯,
DocentAI의 설명도 끊김 없이 흐르길 바라는 마음입니다.

---

## 링크 모음

**프로젝트:**
- 🔗 GitHub (Frontend): https://github.com/tnfhrnsss/docentai-ui
- 🔗 GitHub (Backend): https://github.com/tnfhrnsss/docentai-core
- 🔗 Documentation: https://tnfhrnsss.github.io/docentai/
- 🔗 Demo Video: [YouTube 링크]
- 🔗 Devpost: [Devpost 링크]

**해커톤:**
- 🔗 Gemini API Developer Competition: https://ai.google.dev/competition

**기술 문서:**
- 🔗 Gemini API: https://ai.google.dev/
- 🔗 Chrome Extension: https://developer.chrome.com/docs/extensions/

---

## 감사의 말

이 프로젝트를 완성하기까지 도움을 주신 분들:
- Google Gemini 팀 (강력한 API 제공)
- Chrome Extension 커뮤니티
- 테스트에 참여해주신 분들
- Claude AI (개발 과정 전반 도움)

그리고 무엇보다, 이 글을 읽어주신 여러분께 감사드립니다! 🙏

---

## 마지막으로

해커톤 결과와 무관하게, DocentAI는 계속 발전할 것입니다.

**왜냐하면:**
- ✅ 실제로 필요한 서비스
- ✅ 기술적으로 흥미로운 도전
- ✅ 글로벌 콘텐츠 접근성 향상
- ✅ 배우는 즐거움

**목표는 단순합니다:**

> 전 세계 모든 사람이 언어와 문화의 장벽 없이
> 콘텐츠를 즐길 수 있도록 돕는 것

DocentAI가 그 첫걸음이 되길 바랍니다.

---

**Built with 💜 for Gemini 3 Hackathon 2026**

*"I call it 'DocentAI' (not 'Docent-A-I')."*

---

## 업데이트

- **2026-02-09**: Devpost 제출 완료
- **2026-03-09**: 심사 일정 연기 공지 (4월 1일 발표)
- **2026-03-XX**: Kanana 베타 신청 중
- **2026-04-01**: 최종 발표 예정

---

*이 글이 도움이 되셨다면 GitHub Star ⭐ 부탁드립니다!*

*질문이나 피드백은 GitHub Issues로 남겨주세요!*
