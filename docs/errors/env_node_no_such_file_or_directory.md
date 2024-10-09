---
layout: post
title: env node No such file or directory 해결
date: 2024-10-09 11:10:00
last_modified_at : 2024-10-09 11:10:00
parent: Errors
has_children: false
nav_exclude: true
tags: [node, yarn]
description: env node No such file or directory
--- 


# 환경

- mac os
- homebrew 4.4.0
- nvm 0.40.1
- node 16.0.0
- yarn 1.22.22
- brew로 설치한 경우 버전 알아내기
    - brew list --versions nvm
- brew로 nvm 설치하고 zshrc 파일 생성하고 경로 설정
    
    ```bash
    brew install nvm
    mkdir ~/.nvm
    
    vi ~/.zshrc
    
    export NVM_DIR=~/.nvm
    source $(brew --prefix nvm)/nvm.sh
    
    source ~/.zshrc
    ```
    
    - ✅ 꼭 **source ~/.zshrc** 할 것

# 현상

- react ui 프로젝트를 yarn start dev를 통해 실행하는 상황
- env node No such file or directory  에러가 나면서 실행이 안된다.
- pc재부팅하기 전까진 잘 되었는데, 재부팅하고 나서 다시 안되는 상황

# 원인

- 터미널 실행한 세션에서는 node위치를 모르는 것으로 판단된다.
- which node하면 아무것도 안나옴 😭

# 해결

- 이전 터미널 세션에서 yarn start한 것으로 보여서
- source ~/.zshrc를 다시 해주니 잘된다.