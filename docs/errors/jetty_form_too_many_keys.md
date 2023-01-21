---
layout: post
title: jetty - Form too many keys
date: 2015-09-09 13:07:45
last_modified_at : 2023-01-21 10:13:00
parent: Errors
has_children: false
nav_exclude: true
---

## error log

```prolog
[11-27 14:21:38.885] ERROR [qtp1325124186-21] [s.e.u.b.w.c.LoggingExceptionResolver.resolveException]   30 - Handler execution resulted in exception
java.lang.IllegalStateException: Form too many keys
    at org.eclipse.jetty.util.UrlEncoded.decodeUtf8To(UrlEncoded.java:526)
    at org.eclipse.jetty.util.UrlEncoded.decodeTo(UrlEncoded.java:641)
    at org.eclipse.jetty.server.Request.extractFormParameters(Request.java:372)
    at org.eclipse.jetty.server.Request.extractContentParameters(Request.java:304)
    at org.eclipse.jetty.server.Request.extractParameters(Request.java:258)
    at org.eclipse.jetty.server.Request.getParameter(Request.java:827)
    at org.springframework.web.servlet.mvc.AbstractController.handleRequest(AbstractController.java:153)
    at org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter.handle(SimpleControllerHandlerAdapter.java:48)
    at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:919)
    at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:851)
    at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:953)
    at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:855)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:707)
    at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:829)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
    at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:812)
    at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:587)
    at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:143)
    at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:577)
    at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:223)
    at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1127)
    at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:515)
    at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:185)
    at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1061)
    at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:141)
    at org.eclipse.jetty.server.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:215)
    at org.eclipse.jetty.server.handler.HandlerCollection.handle(HandlerCollection.java:110)
    at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:97)
    at org.eclipse.jetty.server.Server.handle(Server.java:499)
    at org.eclipse.jetty.server.HttpChannel.handle(HttpChannel.java:311)
    at org.eclipse.jetty.server.HttpConnection.onFillable(HttpConnection.java:258)
    at org.eclipse.jetty.io.AbstractConnection$2.run(AbstractConnection.java:544)
    at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:635)
    at org.eclipse.jetty.util.thread.QueuedThreadPool$3.run(QueuedThreadPool.java:555)
    at java.lang.Thread.run(Thread.java:748)
```

## cause

- jetty의 파라미터 key는 최대 1000개가 디폴트.

## problem

jetty의 org.eclipse.jetty.server를 선언한 클래스에 setMaxFormKey(숫자) 를 추가한다.