---
layout: post
title: Claude Code에서 "Interrupted · What should Claude do instead?" 해결하기
parent: Claude
grand_parent: AI
nav_exclude: true
categories: AI
tags: claude  
date: 2026-03-18 23:44:00
last_modified_at : 2026-03-18 23:44:00
---


Claude Code를 사용하다 보면 갑자기 **"Interrupted · What should Claude do instead?"** 라는 메시지가 뜨면서 작업이 중단되는 경우가 있다.

---

## 원인

### 도구 실행 권한 거부 반복

Explorer 에이전트가 코드베이스를 탐색하면서 **음악 디렉토리나 사진, 문서 디렉토리등 전혀 코드와 관련없는 폴더나 프로그램**에 접근하려고 했고, 
그때 마다 계속 "허용 안함"을 선택했다.

에이전트 입장에서는 탐색할 경로가 계속 거부되니 더 이상 진행이 불가능해져서 중단된 것이고, 그게 학습되어서 Explorer할 때마다 "What should Claude do instead?"라고 물어본 것이라고 한다.

---

## 대처 방법

### 즉각적인 대처

- 다시 새로운 터미널을 열고 클로드 실행하면, 처음에만 문제없이 실행되나, 그 이후로는 같은 현상이 난다.

### CLAUDE.md에 탐색 범위 제한 규칙 추가

프로젝트의 `.claude/CLAUDE.md` 파일에 다음 내용을 추가한다:

```markdown
## 탐색 범위 제한

- 파일 탐색, 검색, 읽기는 반드시 이 프로젝트 디렉토리 안에서만 수행할 것.
- 다른 프로젝트나 상위 디렉토리에 절대 접근하지 말 것.
- Agent(Explorer 등)도 동일하게 프로젝트 범위 내에서만 동작할 것.
```

이 규칙은 "이 프로젝트 디렉토리"라고 상대적으로 표현했기 때문에, **다른 프로젝트에도 그대로 복사해서 넣으면 된다.** 각 프로젝트에서 자동으로 해당 프로젝트 경로로 인식한다.

--> 하지만 **비추** . 왜냐하면 뭔가 능동적으로 외부 jar도 봐야하고, 레거시 코드도 봐야한다면, 제한이 있어서 작업할 수 없다고 나온다. 

---

## 참고: .claudeignore와의 차이

| | `.claudeignore` | `CLAUDE.md` 탐색 범위 제한 |
|---|---|---|
| **용도** | 프로젝트 **안의** 특정 파일/디렉토리 제외 | 프로젝트 **밖으로** 나가는 것을 방지 |
| **문법** | `.gitignore`와 동일 | 자연어 규칙 |
| **적용 대상** | 파일 탐색, 읽기 | Agent를 포함한 모든 도구 |

`.claudeignore`는 프로젝트 내부에서 `node_modules/`나 빌드 산출물 같은 불필요한 파일을 제외할 때 사용하고, 프로젝트 외부 접근 차단은 `CLAUDE.md`에 규칙을 추가하는 것이 효과적이라고 한다!


## 근본적인 해결: 재설치

```
npm uninstall -g u/anthropic-ai/claude-code # uninstall claude installed via npm

curl -fsSL https://claude.ai/install.sh | bash

echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc 
```

- 로그인 로그아웃도 해보고
- 캐시도 지워보고
- 컨텍스트 길이 때문인가? 하고 의심도 하고
- /doctor도 하고
- 다 해봤지만 결국, 재설치

---

## 정리

- Explorer가 다른 프로젝트에 접근하려 할 때 "허용 안함"을 반복하면 중단이 발생한다.
- 재설치하자

## Reference
- [interrupted_what_should_claude_do_instead](https://www.reddit.com/r/ClaudeCode/comments/1rwx472/interrupted_what_should_claude_do_instead/)
