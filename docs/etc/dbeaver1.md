---
layout: post
title: how to connect elasticsearch in dbeaver
date: 2022-06-18 22:50:00
last_modified_at : 2022-06-18 22:50:00
parent: Etc
has_children: false
nav_exclude: true
tags: [dbeaver, elasticsearch, es]
---

## 1. 메뉴 [데이터베이스] → [새 데이터베이스 연결]

![dbeaver.png](../img/dbeaver/dbeaver.png)

## 2. host, port, auth정보 입력하고 [Test Connection] 클릭

![dbeaver1.png](../img/dbeaver/dbeaver1.png)

## 3. 드라이브가 없는 경우, Download버튼을 통해 다운받습니다.

![dbeaver2.png](../img/dbeaver/dbeaver2.png)

## 4. 설치한 es버전은 7.6.2인데, jdbc드라이버는 7.8.1이여서 발생한 에러

![dbeaver3.png](../img/dbeaver/dbeaver3.png)

```jsx
This version of the JDBC driver is only compatible with Elasticsearch version 7.9 
or newer; attempting to connect to a server version 7.6.2
```


## 5. elasticsearch 버전에 맞는 jdbc드라이버를 다운받습니다

[Past Releases of Elastic Stack Software](https://www.elastic.co/kr/downloads/past-releases#jdbc-client)

## 6. 그럼 다시 test connection을 통해서 driver를 다운받은걸 교체합니다.

![dbeaver4.png](../img/dbeaver/dbeaver4.png)


[Edit Driver Settings]를 눌러서 교체

![dbeaver5.png](../img/dbeaver/dbeaver5.png)

7.9.1은 Delete하고

Add File을 통해서 다운받은 jar파일로 지정합니다

그리고 다시 Test Connection 클릭하면 성공

![dbeaver6.png](../img/dbeaver/dbeaver6.png)

Dbeaver에 connection 추가한것 확인하고 + 버튼으로 테이블 확인합니다.

![dbeaver7.png](../img/dbeaver/dbeaver7.png)

```jsx
current license is non-compliant for [jdbc]
```

## 7. 에러..에러.......발생

## 결론
basic버전은 jdbc지원 안한다는..플래티넘부터 지원한다고 함. 
슬픈 결론

[programador clic](https://programmerclick.com/article/77401800833/)