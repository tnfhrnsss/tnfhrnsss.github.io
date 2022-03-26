---
layout: post
title: upsource resource제한
date: 2022-03-26 18:01:45
last_modified_at : 2022-03-26 18:01:45
parent: Home
# parent: Etc
has_children: false
nav_order: 1
nav_exclude: false
tags: [upsource]
---


회사에 외주인력도 투입되면서, 리소스 접근제한이 필요하게 되었습니다.

구글에서 업소스 리소스 제한하는 방법을 찾아봤는데요, 

못찾아서, role과 group과 project단위로 user의 권한을 제한하는 방법으로 설정했습니다.

## Project Add

![upsource1](../img/upsource1.png)

Project 메뉴에서 + 버튼을 통해 프로젝트를 추가합니다.

![upsource2](../img/upsource2.png)

Project화면에 등록한 프로젝트가 보이고, 우측에 설정 아이콘을 선택합니다.

### Project > Settings

![upsource3](../img/upsource3.png)

[Move resource here...] 버튼을 클릭해서, user별로 제한할 프로젝트들을 추가합니다.

### Project > Access

프로젝트에 설정한 리소스들에 접근할 group이나 role을 지정해줍니다.

![upsource4](../img/upsource4.png)

[Grant role..] 버튼을 클릭하면, Role, user, group을 선택할 수 있습니다.

### Role별로 업소스 기능 사용을 제한할 수 있습니다.

![upsource5](../img/upsource5.png)

roles메뉴에 가면 Hub 또는 Upsource 어플리케이션별 기능 제한을 할 수 있습니다.

저는 일반 개발자 user에는 코드리뷰 기능외에는 모두 막았습니다.

업소스 사업을 접으면서, 버그에 대한 조치는 이제 없을 듯합니다.

백업을 잘하고, 백업으로 롤백하는 것이 최선일 듯합니다.

오픈소스로 해주면 좋을꺼 같아요

여러가지 커스터마이징하고 싶은 기능도 있고, 수정하고 싶은 버그도 많은 것 같고..

특히 카산드라는 장애가 많이 발생하네요.

암튼 저렇게 설정하고 일반 개발자 계정으로 로그인한 화면입니다.

![upsource6](../img/upsource6.png)

전체 프로젝트가 150개 정도 되는데요. user가 속한 role에 연결된 프로젝트들만 보이고 있습니다.