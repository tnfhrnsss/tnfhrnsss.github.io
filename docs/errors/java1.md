---
layout: default
title: header contains multiple values...
parent: Errors
has_children: false
nav_exclude: true
---

# header contains multiple values '*, *', but only one is allowed

header contains multiple values '*, *', but only one is allowed

라고 크롬콘솔에 찍힐때

response.addheader를 똑같은 값으로 두번한건 아닌지 체크필요. 중복선언시 저런 에러난다.