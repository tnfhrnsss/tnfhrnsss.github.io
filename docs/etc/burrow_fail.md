---
layout: post
title: burrow Failed to compile TemplateOpen
date: 2022-03-29 21:34:00
last_modified_at : 2022-03-29 21:34:00
parent: Etc
has_children: false
nav_exclude: true
tags: [kafka, burrow]
---


### burrow Failed to compile TemplateOpen [recovered]

/root/go/bin/Burrow --config-dir=/usr/local/Burrow/config로 시작하려고 하면 실패나는 것

```
[root@victory-dev-105 config]# cat burrow.out | more
panic: Failed to compile TemplateOpen [recovered]
        panic: Failed to compile TemplateOpen [recovered]
        panic: Failed to compile TemplateOpen

goroutine 1 [running]:
main.handleExit()
        /usr/local/Burrow/main.go:64
        /usr/local/go/src/runtime/panic.go:1038 +0x215
go.uber.org/zap/zapcore.(*CheckedEntry).Write
        /root/go/pkg/mod/go.uber.org/zap@v1.20.0/zapcore/entry.go:232
github.com/linkedin/Burrow/core.configureCoordinators.func1()
        /usr/local/Burrow/core/burrow.go:97 +0x56
panic({0xb3c200, 0xc0001b3a10})
        /usr/local/go/src/runtime/panic.go:1038 +0x215
go.uber.org/zap/zapcore.(*CheckedEntry).Write
        /root/go/pkg/mod/go.uber.org/zap@v1.20.0/zapcore/entry.go:232 +0x446
go.uber.org/zap.(*Logger).Panic
        /root/go/pkg/mod/go.uber.org/zap@v1.20.0/logger.go:230 +0x59
github.com/linkedin/Burrow/core/internal/notifier.(*Coordinator).Configure(0xc000184880)
        /usr/local/Burrow/core/internal/notifier/coordinator.go:217 +0xd91
github.com/linkedin/Burrow/core.configureCoordinators
        /usr/local/Burrow/core/burrow.go:104 +0xc2
github.com/linkedin/Burrow/core.Start(0xc0003288a0, 0xc0001c3ef0)
        /usr/local/Burrow/core/burrow.go:152 +0x3fb
main.main()
        /usr/local/Burrow/main.go:115 +0x4f3
```

### try~

https://github.com/linkedin/Burrow/issues/414에서 시키는대로 burrow.toml에서 `template-open` 부분 수정

기본 파일이 conf/로 되어있어서 경로변경했는데도 아래 에러 발생

```
panic: no url-close specified [recovered]
        panic: no url-close specified [recovered]
        panic: no url-close specified

goroutine 1 [running]:
main.handleExit()
        /usr/local/Burrow/main.go:64 +0xfe
panic({0xb3c200, 0xc000292a70})
        /usr/local/go/src/runtime/panic.go:1038
        /root/go/pkg/mod/go.uber.org/zap@v1.20.0/logger.go:230 +0x59
github.com/linkedin/Burrow/core.configureCoordinators.func1()
        /usr/local/Burrow/core/burrow.go:97
        /usr/local/go/src/runtime/panic.go:1038 +0x215
go.uber.org/zap/zapcore.(*CheckedEntry).Write
        /root/go/pkg/mod/go.uber.org/zap@v1.20.0/logger.go:230 +0x59
github.com/linkedin/Burrow/core/internal/notifier.(*HTTPNotifier).Configure
        /usr/local/Burrow/core/internal/notifier/http.go:78 +0x64b
github.com/linkedin/Burrow/core/internal/notifier.(*Coordinator).Configure
        /usr/local/Burrow/core/internal/notifier/coordinator.go:232 +0x7d1
github.com/linkedin/Burrow/core.configureCoordinators
        /usr/local/Burrow/core/burrow.go:104 +0xc2
github.com/linkedin/Burrow/core.Start
        /usr/local/Burrow/core/burrow.go:152 +0x3fb
main.main()
        /usr/local/Burrow/main.go:115 +0x4f3
```

### solved

just delete this part -> [notifier.default]

다시 nuhup으로 시작

nohup /root/go/bin/Burrow --config-dir=/usr/local/Burrow/config 1> /dev/null 2>&1 &

### reference

[[Kafka] install Burrow to linux (centos)](https://jundol.me/m/147)

[[Kafka] 카프카 Burrow 설치 (카프카 모니터링)](https://veneas.tistory.com/entry/Kafka-%EC%B9%B4%ED%94%84%EC%B9%B4-Burrow-%EC%84%A4%EC%B9%98-%EC%B9%B4%ED%94%84%EC%B9%B4-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81)