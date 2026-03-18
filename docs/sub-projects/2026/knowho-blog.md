# 보이스피싱이 무서워서 앱을 만들었습니다 — Knowho 개발기

> Swift 한 줄도 모르던 내가 iOS 앱을 만들어 앱스토어에 올리기까지

---

## 시작은 불편함이었어요

요즘 보이스피싱 전화가 개인 핸드폰 번호로 오는 경우가 많아졌어요. 후후 같은 앱에도 안 잡히는 번호들이요. 대출 광고인데 개인 번호로 오는 거 보면 진짜 구분이 안 되더라고요.

그래서 저는 아예 회사 분들 번호를 연락처에 저장하지 않기로 했어요. 연락처에 저장하면 카카오톡에 자동으로 연동될 수도 있고, 그게 또 불편했거든요.

대신 고객사 담당자 번호는 메모장에 따로 적어뒀어요. 전화가 오면 메모장 뒤져서 누군지 확인하는 식으로요.

근데 이게 생각보다 너무 불편한 거예요. 전화 오면 빨리 받아야 하는데 메모장 열고 검색하고... 그러다 문득 이런 생각이 들었어요.

**"메모장에 적어둔 번호로 전화 오면 자동으로 알려주면 안 되나?"**

찾아봤는데 그런 앱이 없더라고요. 그래서 직접 만들기로 했습니다.

---

## 아이폰이라 더 복잡했어요

처음엔 쉽게 생각했어요. 메모앱 읽어서 번호 매칭하면 되는 거 아닌가? 했는데, 아이폰은 그게 안 됩니다.

iOS는 샌드박스 구조라 다른 앱이 메모앱 데이터를 직접 읽는 게 불가능해요. 애플이 보안상 막아놨거든요.

한참 찾아보다가 결국 방법을 찾았어요.

```
메모앱에서 공유(내보내기) → 내 앱으로 가져오기 → 앱 내부에 저장
→ Call Directory Extension으로 iOS 전화 시스템에 등록
→ 전화 오면 이름 표시
```

핵심은 **Call Directory Extension** 이에요. 애플이 공식으로 제공하는 API인데, 번호-이름 목록을 iOS 전화 시스템에 등록해두면 전화 왔을 때 자동으로 이름을 표시해줘요. 후스콜, 리멤버 같은 앱들도 다 이 방식을 씁니다.

---

## 기술 스택

| 항목 | 내용 |
|------|------|
| 언어 | Swift |
| UI | SwiftUI |
| 핵심 API | CallKit (Call Directory Extension) |
| 데이터 저장 | App Groups + UserDefaults |
| 아키텍처 | MVVM |

외부 라이브러리는 하나도 안 썼어요. 앱 컨셉 자체가 "완전 로컬, 서버 없음, 광고 없음"이라서 최대한 가볍게 가고 싶었거든요.

---

## 구현하면서 힘들었던 것들

### 1. Call Directory Extension과 메인 앱 데이터 공유

가장 골치 아팠던 부분이에요.

Call Directory Extension은 메인 앱과 **별도 프로세스**로 동작해요. 그래서 메인 앱에서 저장한 데이터를 Extension이 읽으려면 일반 UserDefaults로는 안 되고, **App Groups**라는 걸 설정해야 해요.

```swift
// 일반 UserDefaults — Extension에서 못 읽음 (❌)
UserDefaults.standard.set(data, forKey: "contacts")

// App Groups UserDefaults — Extension에서 읽을 수 있음 (✅)
let shared = UserDefaults(suiteName: "group.com.knowholab.knowho")
shared?.set(data, forKey: "contacts")
```

Xcode에서 메인 앱이랑 Extension 둘 다 같은 App Group으로 설정해줘야 하고, 이걸 모르면 데이터가 전달이 안 돼서 한참 헤맸어요.

---

### 2. 전화번호 E.164 형식 변환

CallKit은 전화번호를 **E.164 국제 형식의 Int64**로 요구해요. 그냥 문자열로 넣으면 안 돼요.

한국 번호 기준으로 이렇게 변환해야 해요.

```swift
func toE164(_ number: String) -> Int64? {
    let digits = number.filter { $0.isNumber }
    
    if digits.hasPrefix("0") {
        // 앞자리 0 제거 후 국가코드 82 추가
        let international = "82" + digits.dropFirst(1)
        return Int64(international)
    }
    return Int64(digits)
}

// 010-1234-5678 → 821012345678
// 02-123-4567   → 8221234567
```

그리고 CallKit에 번호를 등록할 때 **오름차순 정렬이 필수**예요. 정렬 안 하면 Extension이 오류를 뱉어요. 처음엔 이걸 몰라서 왜 안 되나 한참 디버깅했습니다.

```swift
// 오름차순 정렬 필수!
let sorted = contacts.sorted { $0.phoneNumber < $1.phoneNumber }

for contact in sorted {
    context.addIdentificationEntry(
        withNextSequentialPhoneNumber: contact.phoneNumber,
        label: contact.name
    )
}
```

---

### 3. Extension 업데이트가 실시간이 아님

Call Directory Extension은 "실시간 검색" 방식이 아니라 **"사전 등록"** 방식이에요.

메모를 새로 불러오거나 연락처를 수정하면, "전화에 적용" 버튼을 눌러서 Extension을 수동으로 갱신해줘야 해요.

```swift
CXCallDirectoryManager.sharedInstance.reloadExtension(
    withIdentifier: "com.knowholab.knowho.CallDirectory"
) { error in
    if let error = error {
        print("Extension 갱신 실패: \(error)")
    }
}
```

처음엔 저장하면 바로 반영되는 줄 알았는데, 이 코드를 호출해줘야 iOS가 Extension을 다시 읽어요. 이것도 한참 헤맸던 부분이에요.

---

### 4. 시뮬레이터에서 테스트 불가

Call Directory Extension은 **실기기에서만 동작**해요. 시뮬레이터에서는 전화 자체가 안 되니까요.

개발하면서 UI는 시뮬레이터로 확인하고, 실제 전화 수신 테스트는 계속 실기기로 해야 해서 번거로웠어요. 맥이랑 아이폰 USB 연결하고 빌드하고 테스트하고를 반복했습니다.

---

## 앱 이름 — Knowho

앱 이름 짓는 게 생각보다 오래 걸렸어요.

처음엔 "Ring a Bell?", "Do I Know You?" 같은 문장형을 생각했는데, 앱스토어에 비슷한 이름이 있거나 너무 길었어요.

결국 **Know + Who** 를 합친 **Knowho** 로 결정했어요. "누군지 알다"는 의미가 직관적이고, 짧고 기억하기도 쉽고요. 한국어로 읽으면 "노후"라는 발음도 왠지 친근하게 느껴져서요 😄

앱스토어에 동일한 이름이 없는 것도 확인했어요.

---

## 개인정보 처리방침도 직접 만들었어요

앱스토어 심사에 개인정보 처리방침 URL이 필수예요.

서버도 없고 수집하는 데이터도 없는 앱이라 내용은 단순한데, URL이 있어야 하니까 GitHub Pages로 간단하게 만들었어요.

별도 public repo 만들어서 `index.html` 올리고, GitHub Pages 설정하면 URL이 생겨요. 비용 0원이고 관리도 편해요.

> https://tnfhrnsss.github.io/knowho-privacy/

한국어/영어 전환 버튼도 넣었어요.

---

## Claude Code로 개발했어요

Swift를 처음 써봤는데 Claude Code가 없었으면 진짜 못 했을 것 같아요.

터미널에서 `claude` 명령어로 실행하면 프로젝트 전체 파일을 읽고 코드를 직접 작성해줘요. "Call Directory Extension 구현해줘", "온보딩 화면 SwiftUI로 만들어줘" 이런 식으로 대화하면서 개발했어요.

IntelliJ의 AI 플러그인처럼 Xcode 안에 붙는 건 아니고, 터미널 + Xcode를 나란히 띄워놓고 작업하는 방식이에요. 처음엔 어색했는데 익숙해지니까 생산성이 꽤 좋았어요.

---

## 결과물

### 주요 기능
- 메모앱 공유 → 자동 번호 파싱 및 등록
- 텍스트 직접 붙여넣기
- 전화 수신 시 이름 자동 표시
- 등록된 연락처 CSV 내보내기
- 완전 로컬 처리 (서버 전송 없음)

### 앱 구조
```
Knowho (메인 앱)
└── CallDirectoryExtension (발신자 표시)
```

---

## 아직 남은 것들

- [ ] 앱스토어 제출 및 심사
- [ ] 다국어 처리 (한국어 외 영어 등)
- [ ] Watch 앱 (전화 왔을 때 Watch에서도 이름 표시는 메인 앱만 있으면 자동으로 됨)

---

## 마치며

불편함을 해결하려고 시작한 게 어느새 앱스토어 출시 직전까지 왔어요.

Swift를 처음 써봤고, iOS 개발도 처음이었는데 생각보다 할 만했어요. 물론 Call Directory Extension 같은 부분은 문서가 적어서 헤매기도 많이 했지만요.

**"이런 앱 있으면 좋겠다"** 싶은 게 있으면 그냥 만들어보는 게 최고인 것 같아요. 완벽하지 않아도 괜찮아요. 일단 내가 쓸 수 있으면 된 거니까요.

곧 앱스토어에 올라가면 또 후기 들고 올게요! 🙂

---

*앱 이름: Knowho*
*플랫폼: iOS*
*가격: $0.99*
