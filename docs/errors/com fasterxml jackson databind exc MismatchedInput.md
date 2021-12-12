---
layout: default
title: com.fasterxml.jackson.databind.exc.MismatchedInputException
parent: Errors
has_children: false
nav_exclude: true
---

# com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot construct instance of *** (although at least one Creator exists): no String-argument constructor/factory method to deserialize from String value ('{

[에러로그]

com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot construct instance of *** (although at least one Creator exists): no String-argument constructor/factory method to deserialize from String value ('{

[원인]

구글링해보면, 생성자를 생성해주거나 @JsonCreator를 주라고 되어있지만,,,

objectMapper.readValue 하는 class에 맞지 않아서 발생.

string타입인데 array로 받아서 발생한 오류..