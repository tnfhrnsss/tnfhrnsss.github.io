---
layout: post
title: Promise rejected SyntaxError Unexpected end of input
date: 2022-08-16 21:37:00
last_modified_at : 2022-08-16 21:37:00
parent: Errors
has_children: false
nav_exclude: true
tags: [vuejs, vue]
---

## problem

- api를 get방식으로 호출해서 데이터를 리턴받아 화면에 표시하는 간단한 작업이다.
- fetch를 통해서 호출하면 개발자도구로 보면 200인데도, response.ok가 false로 리턴되어있다. (response.ok는 400, 500일 때 false로 됨)

```jsx
fetch('http://localhost:7070/test', {
        method: 'GET',
        mode: 'no-cors',
        credentials: 'same-origin',
        headers: {
          'Content-Type': 'application/json',
          Accept: "application/json",
          "Access-Control-Allow-Origin": "*",
        }
      }).then((response) => {
            console.log(response.json());
        if (response.ok) {
              return response.json();
            } else {
              if (!response.ok) {
                console.log('status --'+response.status);
              }
            }
          })
```

### try. response를 ok여부와 상관없이 json()

```prolog
Scheduler.vue?d1c4:53 **Promise {<rejected>: SyntaxError: Unexpected end of input**
    at eval (webpack-internal:///./node_modules/cache-loader/d…}[[Prototype]]: Promise[[PromiseState]]: "rejected"[[PromiseResult]]: SyntaxError: Unexpected end of input
    at eval (webpack-internal:///./node_modules/cache-loader/dist/cjs.js?!./node_modules/babel-loader/lib/index.js!./node_modules/cache-loader/dist/cjs.js?!./node_modules/vue-loader-v16/dist/index.js?!./src/components/survey/Scheduler.vue?vue&type=script&lang=js:31:30)message: "Unexpected end of input"stack: "SyntaxError: Unexpected end of input\n    at eval (webpack-internal:///./node_modules/cache-loader/dist/cjs.js?!./node_modules/babel-loader/lib/index.js!./node_modules/cache-loader/dist/cjs.js?!./node_modules/vue-loader-v16/dist/index.js?!./src/components/survey/Scheduler.vue?vue&type=script&lang=js:31:30)"[[Prototype]]: Error
Scheduler.vue?d1c4:61 status --0
Scheduler.vue?d1c4:80 SyntaxError: Unexpected end of input (at Scheduler.vue?d1c4:64:1)
    at eval (Scheduler.vue?d1c4:64:1)
Scheduler.vue?d1c4:53 Uncaught (in promise) SyntaxError: Unexpected end of input (at Scheduler.vue?d1c4:53:1)
    at eval (Scheduler.vue?d1c4:53:1)
eval @ Scheduler.vue?d1c4:53
```

## cause

- 구글링 해보면 원인은 다양한 것 같지만, cors에 대한 처리를 해주면 된다.

## resolve

- 백엔드가 spring boot로 되어있어서 cors에 대한 config bean에 ignore path로 등록해줬다.