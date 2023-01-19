---
layout: post
title: ConnectionClosedException
date: 2023-01-19 21:25:00
last_modified_at : 2023-01-19 21:25:00
parent: Nginx
grand_parent: Msa
has_children: false
nav_exclude: true
tags: [nginx]
---

## problem

- 클라이언트에서 전송한 첨부파일을 서버에서 다운로드 할 때 아래 오류 발생하면서 파일을 못갖고 오는 현상

## nginx log

```prolog

upstream prematurely closed connection while reading upstream, 
client: 172.16.100.5, server: localhost, 
request: "GET /attach/download HTTP/1.1", 
upstream: "http://ip/whhC1zokhJ4h5WyoW4hyP0/i_57368f5d6b08.jpeg", 
host: "172.16.100.130:8051"
```

## crema feign log

```prolog
10:51:24.377 DEBUG [,] [7-thread-6] .a.t.c.t.f.c.ThirdPartyGatewayFileClient: 72 log             [ThirdPartyGatewayFileClient#download] <--- ERROR ConnectionClosedException: Premature end of Content-Length delimited message body (expected: 151,661; received: 75,091) (31ms)
10:51:24.378 DEBUG [,] [7-thread-6] .a.t.c.t.f.c.ThirdPartyGatewayFileClient: 72 log             [ThirdPartyGatewayFileClient#download] org.apache.http.ConnectionClosedException: Premature end of Content-Length delimited message body (expected: 151,661; received: 75,091)
	at org.apache.http.impl.io.ContentLengthInputStream.read(ContentLengthInputStream.java:178)
	at org.apache.http.conn.EofSensorInputStream.read(EofSensorInputStream.java:135)
	at org.apache.http.conn.EofSensorInputStream.read(EofSensorInputStream.java:148)
	at feign.Util.copy(Util.java:329)
	at feign.Util.toByteArray(Util.java:312)
	at feign.Logger.logAndRebufferResponse(Logger.java:126)
	at feign.slf4j.Slf4jLogger.logAndRebufferResponse(Slf4jLogger.java:61)
	at feign.AsyncResponseHandler.handleResponse(AsyncResponseHandler.java:70)
	at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:141)
	at feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:91)
	at org.springframework.cloud.openfeign.FeignCircuitBreakerInvocationHandler.lambda$asSupplier$1(FeignCircuitBreakerInvocationHandler.java:140)
	at java.util.concurrent.FutureTask.run$$$capture(FutureTask.java:266)
	at java.util.concurrent.FutureTask.run(FutureTask.java)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)

```

## resolve

- 응답을 못받고 끊어질 때 nginx.conf에 아래 설정 추가하면 된다.

```prolog
proxy_read_timeout 300;
proxy_connect_timeout 300;
proxy_send_timeout 300;
```