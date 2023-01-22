---
layout: post
title: org.apache.jasper.JasperException- PWC6349
date: 2023-01-22 20:19:00
last_modified_at : 2023-01-22 20:19:00
parent: Errors
has_children: false
nav_exclude: true
---

## situation

org.apache.jasper.JasperException: PWC6349: Cannot find a java compiler for compilation. If running with JDK 5 or before, Ant or JDT compiler can be used, if the corresponding jars and bridge classes

## error log

```prolog
08-29 08:29:54 550076-246 WARN  o.e.j.s.ServletHandler  :620 doHandle                       
org.apache.jasper.JasperException: PWC6349: Cannot find a java compiler for compilation.  If running with JDK 5 or before, Ant or JDT compiler can be used, if the corresponding jars and bridge classes (org.apache.jasper.compiler.AntJavaCompiler or org.apache.jasper.compiler.JDTJavaCompiler) are included
    at org.apache.jasper.compiler.DefaultErrorHandler.jspError(DefaultErrorHandler.java:92)
    at org.apache.jasper.compiler.ErrorDispatcher.dispatch(ErrorDispatcher.java:378)
    at org.apache.jasper.compiler.ErrorDispatcher.jspError(ErrorDispatcher.java:119)
    at org.apache.jasper.compiler.Compiler.initJavaCompiler(Compiler.java:773)
    at org.apache.jasper.compiler.Compiler.<init>(Compiler.java:140)
    at org.apache.jasper.JspCompilationContext.createCompiler(JspCompilationContext.java:288)
    at org.apache.jasper.JspCompilationContext.compile(JspCompilationContext.java:622)
    at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:375)
    at org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:473)
    at org.apache.jasper.servlet.JspServlet.service(JspServlet.java:377)
    at org.eclipse.jetty.jsp.JettyJspServlet.service(JettyJspServlet.java:103)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
    at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:812)
    at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:587)
    at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:143)
    at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:595)
    at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:223)
    at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1127)
    at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:515)
    at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:185)
    at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1061)
    at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:141)
    at org.eclipse.jetty.server.Dispatcher.forward(Dispatcher.java:191)
    at org.eclipse.jetty.server.Dispatcher.forward(Dispatcher.java:72)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:707)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
    at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:812)
    at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1669)
    at org.eclipse.jetty.websocket.server.WebSocketUpgradeFilter.doFilter(WebSocketUpgradeFilter.java:201)
    at org.eclipse.jetty.servlet.ServletHandler$CachedChain.doFilter(ServletHandler.java:1652)
    at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:585)
    at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:143)
    at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:577)
    at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:223)
    at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1127)
    at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:515)
    at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:185)
    at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1061)
    at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:141)
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

jsp servlet을 호출하려고 할때

## solved

jetty9버전에는 톰캣에서 지원하던 glossary를 통해 하지않기 때문에
jetty-jsp-jdt를 설치해야한다.