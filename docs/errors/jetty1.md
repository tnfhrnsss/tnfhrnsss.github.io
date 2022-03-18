---
layout: post
title: Can't generate resourceBase as part of webapp tmp dir name
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
tags: [jetty, intellij]
---

# Can't generate resourceBase as part of webapp tmp dir name: java.lang.NullPointerException

현상) 인텔리제이에서 application으로 jetty올리는데 에러가 나면서 안올라갈때

오류) [09-29 09:01:17.454] WARN [main] [o.e.j.w.WebInfConfiguration.getCanonicalNameForWebAppTmpDir] 627 - Can't generate resourceBase as part of webapp tmp dir name: java.lang.NullPointerException

[09-29 09:01:17.665] WARN [main] [o.e.j.w.WebAppContext.doStart] 514 - Failed startup of context o.e.j.w.WebAppContext@17695df3{/router,[],null}

java.lang.NullPointerException: null

at org.eclipse.jetty.webapp.WebAppContext.getWebInf(WebAppContext.java:844)

at org.eclipse.jetty.webapp.WebInfConfiguration.findWebInfLibJars(WebInfConfiguration.java:708)

at org.eclipse.jetty.webapp.WebInfConfiguration.findJars(WebInfConfiguration.java:689)

at org.eclipse.jetty.webapp.WebInfConfiguration.preConfigure(WebInfConfiguration.java:128)

at org.eclipse.jetty.webapp.WebAppContext.preConfigure(WebAppContext.java:468)

at org.eclipse.jetty.webapp.WebAppContext.doStart(WebAppContext.java:504)

at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:68)

at org.eclipse.jetty.util.component.ContainerLifeCycle.start(ContainerLifeCycle.java:132)

at org.eclipse.jetty.util.component.ContainerLifeCycle.doStart(ContainerLifeCycle.java:114)

at org.eclipse.jetty.server.handler.AbstractHandler.doStart(AbstractHandler.java:61)

at org.eclipse.jetty.server.handler.ContextHandlerCollection.doStart(ContextHandlerCollection.java:163)

at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:68)

at org.eclipse.jetty.util.component.ContainerLifeCycle.start(ContainerLifeCycle.java:132)

at org.eclipse.jetty.util.component.ContainerLifeCycle.doStart(ContainerLifeCycle.java:114)

at org.eclipse.jetty.server.handler.AbstractHandler.doStart(AbstractHandler.java:61)

at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:68)

at org.eclipse.jetty.util.component.ContainerLifeCycle.start(ContainerLifeCycle.java:132)

at org.eclipse.jetty.server.Server.start(Server.java:387)

at org.eclipse.jetty.util.component.ContainerLifeCycle.doStart(ContainerLifeCycle.java:114)

at org.eclipse.jetty.server.handler.AbstractHandler.doStart(AbstractHandler.java:61)

at org.eclipse.jetty.server.Server.doStart(Server.java:354)

at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:68)

원인)

첫번째로, 이 프로젝트랑 상관없는 path의 디렉토리를 옮겼었는데, 그걸 못찾는다고 나온당...그래서 그건 그냥 .idea폴더 날리고 , 프로젝트 재생성

두번째로, 어플리케이션 올릴때 Working directory에 %MODULE_WORKING_DIR%이렇게 줘야하는데 없어서,,,, 참고로 $MODULE_DIR$ 에서 %MODULE_WORKING_DIR% 이렇게 바뀜