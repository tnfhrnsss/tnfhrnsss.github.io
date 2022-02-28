---
layout: post
title: Tesseract Ocr Study
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Home
# parent: Sub Projects
has_children: false
nav_order: 4
nav_exclude: true
---

# tesseract ocr 공부

기술조사 ) Tesseract-ocr을 사용

순서1) 라이브러리설치에 앞서서, 맥으로 해야하기 때문에 홈브루 설치(https://whitepaek.tistory.com/3)

순서2) tesseract, tesseract-lang설치

> brew install tesseract

> brew install tesseract-lang

순서3) sudo pip3 install pytesseract

sudo pip3 intall Image

순서4) 실행한다.

```bash
peter:~ peter$ tesseract /Users/peter/Downloads/IMG_5814.JPG /Users/peter/Downloads/aaa.txt -l kor
```

순서4-1) 또는 파이썬으로 실행한다.

import pytessearct

import PIL import Image

```bash
print(pytesseract.image_to_string(Image.open('/Users/peter/Downloads/IMG_5814.JPG'), lang='kor'))
```

결과1) 별로 좋지 않은 결과

```bash
이 도 아이티 들으 그도 밸아차

가 소경더 [준공 023

자구 가바바 다자이 이누 그두자구

1611 11\ 1000 너(|'0>030)40)

이 지 성 지음
```

시도2) 이미지를 rotate시켜본다.

```bash
>>> im = Image.open('/Users/peter/Downloads/IMG_5814.JPG')

>>> img3 = im.rotate(90)

>>> print(pytesseract.image_to_string(img3, lang='kor'))
```

결과2) 더 안좋음 ㅠㅜ

```bash
몽|? 을 |? |0

 

도스 사기브 사고    ~ 요일 -………애 : 다

 

00000                                 0 디넌낸트이나 조는

[1

~

야

오어

1오          "

0
```

시도3) tesseract의 train된 lang을 추가해본다.

https://github.com/tesseract-ocr/tessdata/blob/master/kor.traineddata 에서 다운로드받고,

받은 데이터파일을 아래에 추가.

mac으로 terracert 를 설치한 경우, 추가할 위치는 /usr/local/share/tessdata 이다.

결과3) 첫번째에 비해 달라진걸 모르겠다 ㅠㅜ

```bash
人 還 計生 lane Austen ()%+()&1)

 

 

點

『 | 1 도

~) 人 全 一 人 生 人 加

人 크

” > 재한 0 블 세 랐 도 기 ||” :( 총 니 . 내 - 올 를

SKIN 7! 올블

 

이 入 성 지음
```

시도4) 문제를 알았다... 폰트같은 글자 모양이나 길이에 대한 학습이 필요하다는 것을...

시도5) 이미지를 box파일을 만든담에 [https://hello-gg.tistory.com/5](https://hello-gg.tistory.com/5) 에서처럼 하나씩 불러서 인식을 잡아준다. 그리고 학습 후 기존 학습파일에 add해준다. [https://diyworld.tistory.com/114](https://diyworld.tistory.com/114)