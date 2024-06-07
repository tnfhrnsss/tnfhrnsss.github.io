---
layout: post
title: React js 학습 노트 (비기너)
date: 2024-06-07 10:44:00
last_modified_at : 2024-06-07 10:44:00
parent: Youtube
grand_parent: Mooc
nav_exclude: true
---

# Plan

- 유튜브에서 “react js 강의” 라고 검색했다.
- 매일 5~10분 정도의 학습량으로 계획했다.


# 코딩앙마 채널 학습

### 1. 강의 정보

- [강의 URL](https://www.youtube.com/playlist?list=PLZKTXPmaJk8J_fHAzPLH8CJ_HO_M33e7-){:target="_blank"}
- [github 샘플](https://github.com/coding-angma/voca){:target="_blank"}

### 2. 학습내용

- 패키지 구조, hmr (hot module replacement)에 대해 설명
- css 작성법
    - 모듈 css 사용법
    - 파일명.module.css라고 생성하고 스타일 작성
    - 스타일 임포트하고, {styles.box} 이런 식으로 작성한다.
    
    ```jsx
    import styles from "./Hello.module.css";
    
    export default function Hello() {
        return (
            <>
              <div className="{styles.box}"></div>
            </>
        );
    }
    ```
    

- state, useState
    - 동일한 컴포넌트라도 state는 독립적이다. 다른 state에 영향을 미치지 않는다.
    - react 16.8 부터 hook을 사용할 수 있다.

- props
    - properties로 속성값을 의미한다.
    - 값은 전달받은 컴포넌트 내부에서 변경할 수 없다. 그대로 사용해야한다.
    - 변경하고 싶다면, 컴포넌트 안에서 state를 다시 만들어야한다.
        
        ```jsx
        export default function Hello(props) {
            console.log(props)
            const [age, setAge] = useState(props.age); // state로 등록하고 props값을 초기값으로 한다
        
            return (
                <div>
                    <h2>({age})</h2>
                    <button onClick={() => {
                        setAge(age + 1);
                    }}>Change</button>
                </div>
            );
        }
        ```
        
    - 한 컴포넌트가 갖고있는 state를 props로 넘길 수 있다.
- map
    - map으로 반환되는 요소는 jsx로 작성한다.
        
        ```jsx
        <tbody>
          {wordList.map(word => (
              <tr>
                  <td>
                      {word.eng}
                  </td>
                  <td>
                      {word.kor}
                  </td>
              </tr>
          ))}
        </tbody>
        ```
        
    - map.filter 사용
- router
    - <Route **exact** path="/"> path에 정확히 매칭되는 것만일때
        - 하위 패스가 있을땐, <Route path="/day/**:**day"> 이렇게 콜론을 사용한다.
    - <a> 앵크태그대신 리액트에서는 <Link> 태그를 사용한다.
- json-server 사용해서 restapi 호출
    
    ```bash
    json-server --watch ./src/db/data.json --port 3001
    ```
    
- useEffect
    - 어떤 상태값이 바뀌었을 때 동작하게 하는 함수를 작성하는데 사용한다.
    - 렌더링 이후 돔에 반영된 이후 시점, 컴포넌트가 사라지는 시점에 동작한다.
    - 첫번째 매개변수로 함수를 넣고
    - 두번째 매개변수로 의존성 배열을 넣고, 의존성 배열이 변경되었을때만 함수가 실행되도록 할 수 있다.
        - 만약에 빈배열로 하면, 최초 한번만 실행되게 된다.
- custom hook
- 부록: 타입스크립트 적용
    
    ```bash
    npm install typescript @types/node @types/react @types/react-dom @types/jest @types/react-router-dom
    ```
    
    - js파일 → ts, jsx 파일 → tsx로 변경
    - 타입스크립트에서 any를 남발하면 안된다. any를 쓰면 타입스크립트를 쓰는 의미가 없다.
    

### 3. 정리

- 핵심만 빠르고 쉽게 설명해줘서 지루하지 않게 학습할 수 있었다.