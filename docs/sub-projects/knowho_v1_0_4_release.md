---
layout: post
title: Knowho 1.0.4 업데이트 and 유튜브 소개 영상
description: Knowho 1.0.4 릴리즈 노트(사진 번호 추출, 삭제 실행 취소)와 Claude로 만든 유튜브 소개 영상 정리
date: 2026-10-04 10:00:00
last_modified_at: 2026-10-04 10:00:00
parent: Sub Projects
has_children: false
nav_exclude: true
categories: iOS
tags: claude claude-code ios swift vision ocr knowho youtube
---

3월에 출시한 [Knowho](https://apps.apple.com/app/id6760998976){:target="_blank"}의 1.0.4 업데이트 했습니다.  
이번에도 Claude Code와 함께 작업했고, 겸사겸사 유튜브 소개 영상도 다이나믹 모션 그래픽으로 만들어 봤습니다.  

> 처음 만든 이야기는 [Claude Code로 iOS 앱 만들기 – Knowho 바이브코딩 후기](./knowho_with_claude_code.md)에 정리해 두었습니다.

---

## 1.0.4 업데이트 내용

```
 • 사진 속 전화번호 자동 추출 (프리미엄 기능): 사진에서 번호를 찾아 바로 등록할 수 있어요
 • 연락처 삭제 복구 기능: 삭제 후 "실행 취소" 버튼이나 기기 흔들기로 되돌릴 수 있어요
 • 기타 버그 수정
```

릴리즈 노트에는 짧게 적었지만, 실제로는 이것저것 손본 게 꽤 많습니다.

| 구분 | 내용 |
|------|------|
| 사진 번호 추출 (Pro) | 사진 한 장을 기기 안에서 OCR로 읽어 전화번호만 추출 |
| 붙여넣기 개선 | "번호 추출" 버튼 없이 붙여넣으면 바로 추출 |
| 추출 결과 편집 | 이름·번호 직접 수정, 잘못된 번호는 빨간색, 밀어서 삭제 |
| 저장 버튼 | 화면 하단 고정 "N건 저장하고 닫기" + 키보드 위 "확인" 바 |
| 삭제 실행 취소 | 하단 토스트의 "실행 취소" 또는 기기 흔들기 |
| iCloud 백업 | 말풍선 메뉴 → 하단 시트로 변경, 마지막 백업 시간 표시 |
| 버그 수정 | 한 줄에 여러 번호 추출, 이름 뒤 쉼표·하이픈 제거, 1MB 초과 백업 실패 안내 등 |

---

## 사진에서 전화번호 추출하기

### 처음 계획은 "번호 + 이름"

처음에는 사진에서 번호와 이름을 같이 뽑으려고 했습니다.  
Apple의 Vision 프레임워크를 쓰면 서버 없이 기기 안에서 OCR이 되니까,  
"완전 오프라인"이라는 Knowho의 원칙도 지킬 수 있었습니다.  

그런데 Claude와 이야기해 보니 이름 추출은 생각보다 까다로웠습니다.

그래서 **번호만 추출하고, 이름은 마지막에 사용자가 직접 입력**하는 쪽으로 정했습니다.  
번호는 형식이 정해져 있어서 인식률이 높고, 이름은 어차피 사용자가 확인해야 하니까요.  

### 손글씨는 아직 어렵다

테스트하면서 손글씨로 적은 `010-xxxx-xxxx`를 찍어봤는데, 엉뚱한 숫자로 인식되었습니다.  
몇 가지 시뮬레이션 돌려본 후, 가이드 영역으로 빼고 한계점을 드러내는 것으로 결정했습니다. 

---

## 유튜브 소개 영상

앱 소개 영상도 Claude와 함께 만들었습니다.  
영상 편집 툴 없이, **HTML + 애니메이션(GSAP)으로 장면을 만들고 프레임마다 캡처해서 MP4로 렌더링**하는 방식입니다.  

- 장면 구성과 애니메이션: HTML 한 장 (1920×1080, 약 55초)
- 렌더링: 헤드리스 Chrome으로 프레임별 캡처 → ffmpeg로 60fps 인코딩
- 배경음악: Pixabay 무료 음원, 로고 등장 타이밍에 빌드업이 맞도록 시작 지점 조정

### 한국어

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%;">
  <iframe style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"
          src="https://www.youtube.com/embed/dM8Yw8GMOrs"
          title="Knowho 소개 영상" frameborder="0"
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
          allowfullscreen></iframe>
</div>

### English

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%;">
  <iframe style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"
          src="https://www.youtube.com/embed/HJZ_GcBrQ4w"
          title="Knowho Intro Video" frameborder="0"
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
          allowfullscreen></iframe>
</div>

---

## 마무리

출시 후 기능을 하나씩 붙여 가면서 느낀 건,  
갈수록 회귀테스트가 중요하다는 것 ..
테스트 시나리오가 늘어난다는 것 ..

[App Store에서 Knowho 보기](https://apps.apple.com/app/id6760998976){:target="_blank"}
