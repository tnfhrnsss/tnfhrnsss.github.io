---
layout: post
title: Expert One-on-One J2EE Design and Development 번역 정리
date: 2023-02-23 19:57:45
last_modified_at : 2023-02-23 19:57:45
parent: Clipping
has_children: false
---

# 1. ****J2EE Architectures****

- J2EE는 많은 아키텍처를 제공한다. J2EE는 또한 많은 컴포넌트 타입들을 제공한다.(servlet이나 ejb, jsp page, servlet filter등) 그리고 J2EE 어플리케이션 서버는 많은 추가 서비스들을 제공한다.
- 이러한 옵션들은 문제해결에 최고의 선택을 할 수 있게도 하고 또는 위험한 결과를 초래하게 하기도 한다.
- J2EE 개발자는 선택지가 많아서 좋을 수도 있지만, 잘못된 선택으로 부적절하게 적용할 수도 있다.
- 이 챕터에서 J2EE 아키텍처를 개발할 때 좋은 최상위 요건에 대해 논의하고 어떻게 실상황에 녹일 수 있는지 논의하도록 한다.
    - 분산 어플리케이션과 비분산(non-distributed) 어플리케이션 중 어떤 모델이 적합한지 선택하는 방법  