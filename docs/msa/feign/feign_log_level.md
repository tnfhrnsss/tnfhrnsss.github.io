---
layout: post
title: Feign log level
date: 2022-07-04 21:37:00
last_modified_at : 2022-07-04 21:37:00
parent: Feign
grand_parent: Msa
nav_exclude: true
tags: [feign, log]
---

level로 full, basic, none, headers가 있음

## basic

```
07:44:21.783   [ReceptionClient#register] ---> POST [http://mocha/partitions/test/receptions](http://mocha/partitions/test/receptions) HTTP/1.1
07:44:23.872   [ReceptionClient#register] <--- HTTP/1.1 200  (2084ms)
```

## headers

```
07:46:45.695  [ReceptionClient#register] ---> POST [http://mocha/partitions/test/receptions](http://mocha/partitions/test/receptions) HTTP/1.1
07:46:45.695  [ReceptionClient#register] Accept: application/json
07:46:45.696  [ReceptionClient#register] Content-Length: 336
07:46:45.696  [ReceptionClient#register] Content-Type: application/json
07:46:45.696  [ReceptionClient#register] Content-Type: charset=utf8
07:46:45.696  [ReceptionClient#register] ---> END HTTP (336-byte body)
07:46:45.845  [ReceptionClient#register] <--- HTTP/1.1 200  (148ms)
07:46:45.845  [ReceptionClient#register] connection: keep-alive
07:46:45.845  [ReceptionClient#register] content-type: application/json
07:46:45.845  [ReceptionClient#register] date: Thu, 29 Apr 2021 22:46:45 GMT
07:46:45.845  [ReceptionClient#register] keep-alive: timeout=60
07:46:45.845  [ReceptionClient#register] transfer-encoding: chunked
07:46:45.845  [ReceptionClient#register] <--- END HTTP (130-byte body)
```

## full

```
07:49:09.457 [ReceptionClient#register] ---> POST [http://mocha/partitions/test/receptions](http://mocha/partitions/test/receptions) HTTP/1.1
07:49:09.458 [ReceptionClient#register] Accept: application/json
07:49:09.458 [ReceptionClient#register] Content-Length: 336
07:49:09.458 [ReceptionClient#register] Content-Type: application/json
07:49:09.458 [ReceptionClient#register] Content-Type: charset=utf8
07:49:09.458 [ReceptionClient#register]
07:49:09.458 [ReceptionClient#register] {"receptionId":"96d7abbd-1c20-46c3-8931-56649759f17e","customer":{"id":"aaaaaaaa","loginId":"bbbbb","loggedIn":false,"metadata":{"externalId":"bbbbbbbb"}},"appChannelId":"ccccccccc","firstMessage":"네","customType":"customer","topicId":"default-topic","fileMetaIds":[]}
07:49:09.459 [ReceptionClient#register] ---> END HTTP (336-byte body)
07:49:09.640 [ReceptionClient#register] <--- HTTP/1.1 200  (181ms)
07:49:09.641 [ReceptionClient#register] connection: keep-alive
07:49:09.641 [ReceptionClient#register] content-type: application/json
07:49:09.641 [ReceptionClient#register] date: Thu, 29 Apr 2021 22:49:09 GMT
07:49:09.641 [ReceptionClient#register] keep-alive: timeout=60
07:49:09.642 [ReceptionClient#register] transfer-encoding: chunked
07:49:09.642 [ReceptionClient#register]
07:49:09.642 [ReceptionClient#register] {"receptionId":"96d7abbd-1c20-46c3-8931-56649759f17e","customerId":"aaaaaaa","messageId":391668525187072}
07:49:09.642 [ReceptionClient#register] <--- END HTTP (130-byte body)
```