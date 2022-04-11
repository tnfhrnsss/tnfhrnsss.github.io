---
layout: post
title: ios 웹앱 만들기
date: 2022-04-11 21:52:00
last_modified_at : 2022-04-11 21:52:00
parent: Etc
has_children: false
nav_exclude: true
tags: [ios, hybrid app, firebase push]
---

## 목표

firebase로 푸시받는 것 까지 하고 앱으로 배포

## 기본 웹앱 작성

웹앱 코드([https://www.youtube.com/watch?v=JafGypqFvs4](https://www.youtube.com/watch?v=JafGypqFvs4))

## 인증서 작업

- testflight으로 푸시테스트가 가능하려면,  AdHoc Provisioning Profile which will use the production push certificate을 발급받아야 한다. (개발용이면 안됨)
- 일단 fcm을 통한 발송준비용 인증서
    - fcm을 통해서 발송하는 것은 apns를 통해서 하는 인증서 절차보다 간단합니다. apns를 통해 발송할 경우, fcm에서 한 절차보다 몇 가지 더 하면 됩니다.(두개를 공통으로 사용가능)
    - mac의 키체인에서 인증서 만들고
    - **Identifiers app등록**
        - **App ID Prefix는 아무거나 (teamid가 없는것도 됨)**
        - push notification 선택
    - 다 등록하고 다시 상세내용에서 push notification의 edit 화면에서 배포용 인증서 생성
        - 로컬 키체인에서 csr파일 생성 (앱이 다르면, 각각 만들어주는 것이 좋다) - [https://ios-development.tistory.com/247](https://ios-development.tistory.com/247)
        - 그러면 **Certificates, Identifiers & Profiles > Certificate에서 푸시용 배포용 2개만들어지게 됨**
        - 그러면 총 개인용, 푸시용(Apple Push Services로 등록한), 배포용 3개가 됨
        - 앱을 동일 계정으로 여러개 만들 경우는 identifiers app등록하고 csr인증서 연결해주고 key등록해주면 됨
    - Profile은 fcm일땐 생성안해도 됨
    - Keys 생성 (중요!)
        - 나중에 fcm 콘솔에 등록해야 하므로, 다운로드 파일(p8), Key Id 적어야함
- firebase 설정
    - firebase 프로젝트 생성하고 → plist파일을 xcode에 넣고 → xcode 프로젝트에 firebase 라이브러리 추가하고(라이브러리는 firebase messaging 만 선택) → firebase 코드 추가한다.
    - 클라우드 메시지 → **APN 인증 키 추가**
        - apple developer의 Key 등록했을 때 받은, Keys의 Key Id, AuthKey~~.p8
        - team Id는 계정의 팀 아이디를 적는다(인증서랑 무관)
- 이제는 xcode 작업
    - Sigining & Capabilities에 push notification 과 background modes 추가
        - background modes에는 background fetch와 remote notifications 선택
    - info.plist에 속성 추가
        - FirebaseAppDelegateProxyEnabled를 boolean으로 추가하고 NO로 설정

## 푸시 메시지 발송

- 테스트
    - node의 apn모듈로 발송
        - 준비물 : 위에서 인증서 생성시 다운받은 p8파일과 p8파일의 KeyID,  팀ID, 번들ID
        
        [iOS APNs 인증 키 p8 (iOS Key로 푸시 보내기)](https://taesulee.tistory.com/5)
        
    - firebase를 통해서 발송하는 방법도 있음
        - token을 에스프레소로 앱에서 호출하도록 해야함
        - 발송시간을 지금으로 해도 폰으로 오기까지 2~3분정도 걸리는 것 같다
- token들을 서버 API를 통해 등록시키기
    
    Alamofire라이브러리를 pod로 설치해서 post api 
    
    [iOS : Alamofire 를 이용한 API 호출](https://velog.io/@budlebee/iOS-Alamofire-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-API-%ED%98%B8%EC%B6%9C)
    

## 기타

- 앱아이콘 적용([https://appicon.co/](https://appicon.co/))
- 뱃지 : ios는 서버에서 숫자를 발송시켜줘야한다고 함. 앱에선 왔을때 0으로 초기화하는 코드만 두고

## 배포

- 방식 결정 : 배포방식 중 testflight을 사용해보도록 함

[알면 알수록 헷갈리는 IOS 환경 #2 - 앱 배포방식에 대해서 알아보자.](https://www.blueswt.com/124)

- 순서
    - [https://dev-yakuza.posstree.com/ko/react-native/ios-testflight/](https://dev-yakuza.posstree.com/ko/react-native/ios-testflight/) 에 따라 xcode에서 앱 배포
    - xcode의 archive로 여러 앱을 했더니 자꾸 codesign이 다른 키체인껄로 되어서 아래와 같은 에러가 난다.
    
    ```
    App Store Connect Operation Error
    
    No suitable application records were found. 
    Verify your bundle identifier ‘~~~’ is correct.
    ```
    
    —> 여러 가지 방법을 다 해봤지만, 결국 안되어서 Transporter앱을 통해서 업로드했다([https://red-cherry-ring.tistory.com/16](https://red-cherry-ring.tistory.com/16)) —> 성공!!
    
    - 배포전 info.plist파일에 수출규정준수 설정 제외시켜야 함 ([https://jh3786.tistory.com/11](https://jh3786.tistory.com/11))
- 참고
    - [https://testflight.apple.com/](https://testflight.apple.com/)

## reference

[https://dev200ok.blogspot.com/2020/04/ios-notification-push.html](https://dev200ok.blogspot.com/2020/04/ios-notification-push.html)

[https://www.raywenderlich.com/11395893-push-notifications-tutorial-getting-started](https://www.raywenderlich.com/11395893-push-notifications-tutorial-getting-started)

## 기타

[[iOS] 푸시알림 클라우드 메세지 보내기[2] (APNS, 파이어베이스)](https://developer111.tistory.com/43)

- 내 계정이 회사 계정의 팀멤버로 연결되어서, 인증서를 생성할 수 없는 문제가 생겼는데.(팀관리자 계정이 갱신이 안되어서..) - 일단 푸시하려면 구독이 필요해보인다.
- 내 맥은 카탈리나 10.15.7이고 xcode는 12.5로 ios15이상은 시뮬레이터나 폰에도 설치가 안되는 문제가 있다.
    - xcode13버전의 파일을 일부 옮겨와서 폰에 설치하려고 하니, 빌드는 성공했지만 아래오류 발생
    
    ```
    The code signature version is no longer supported. 
    Domain: com.apple.dt.MobileDeviceErrorDomain
    ```
    
    - 해결은 [https://stackoverflow.com/a/68467307](https://stackoverflow.com/a/68467307) 이렇게 -generate-entitlement-der를 추가해줬다.
    - 저걸해주고 나서 fcm연동하니 발생했는데, TARGETS -> select[your project name] -> General -> Frameworks,Libraries,and EmbeddedContent 에 가서 Firebase Analystic인가 하는 라이브러리를 제거해줬더니 해결
- p12파일을 생성하기까지

[iOS) APNs :: 인증서 발급받는 방법 (p.12, pem)](https://babbab2.tistory.com/57)

[[ios] 자바 스프링 서버에서 iOS앱에 푸시 알림 보내기(APNs 개발용, 배포용)](https://developer111.tistory.com/44)

[[iOS] Spring 서버에서 사용할 APNS 인증서 준비](https://scshim.tistory.com/31)