---
layout: post
title: org.apache.jasper.JasperException PWC6033 Error in Javac compilation for JSP
date: 2015-09-09 13:07:45
last_modified_at : 2023-01-20 22:02:45
parent: Errors
has_children: false
nav_exclude: true
---

## problem

- 서비스 올리고나서 브라우저에서 붙으면 아래와 같은 오류만 나고, 페이지를 수정해도 jsp가 컴파일되지 않는

## error log

```prolog
16:25:07.234 [qtp1054507852-19 - /front/main/jsp/view/domainPage.jsp] DEBUG org.eclipse.jetty.webapp.WebAppClassLoader - loaded class com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl from null
16:25:07.235 [qtp1054507852-19 - /front/main/jsp/view/domainPage.jsp] DEBUG org.eclipse.jetty.webapp.WebAppClassLoader - loaded class com.sun.org.apache.xerces.internal.impl.dv.dtd.DTDDVFactoryImpl from null
6월 25, 2018 4:25:07 오후 org.apache.jasper.compiler.Compiler generateClass
심각: Error compiling file: C:\Users\AppData\Local\Temp\jetty-0.0.0.0-7070-webapp-_-any-\jsp\org\apache\jsp\front\main\jsp\view\domainPage_jsp.java
16:25:07.794 [qtp1054507852-19 - /front/main/jsp/view/domainPage.jsp] DEBUG org.eclipse.jetty.servlet.ServletHandler -
org.apache.jasper.JasperException: PWC6033: Error in Javac compilation for JSP

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
illegal start of type

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
= expected

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
';' expected

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
<identifier> expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
illegal start of type

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
= expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
';' expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
<identifier> expected

PWC6199: Generated servlet error:
illegal start of type

PWC6199: Generated servlet error:
= expected

PWC6199: Generated servlet error:
';' expected

PWC6199: Generated servlet error:
<identifier> expected

PWC6199: Generated servlet error:
illegal start of type

PWC6199: Generated servlet error:
= expected

PWC6199: Generated servlet error:
';' expected

PWC6199: Generated servlet error:
<identifier> expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
illegal start of type

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
= expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
';' expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
<identifier> expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
illegal start of type

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
= expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
';' expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
> expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
< expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
illegal start of expression

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
'(' expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
<identifier> expected

PWC6197: An error occurred at line: 129 in the jsp file: /front/main/jsp/view/../include/header.jspf
PWC6199: Generated servlet error:
package java.util.function does not exist

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class getHttpServletMapping
location: interface javax.servlet.http.HttpServletRequest

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class newPushBuilder
location: interface javax.servlet.http.HttpServletRequest

PWC6199: Generated servlet error:
C:\Users\.m2\repository\org\jboss\spec\javax\servlet\jboss-servlet-api_4.0_spec\1.0.0.Final\jboss-servlet-api_4.0_spec-1.0.0.Final.jar(javax/servlet/http/HttpUpgradeHandler.class): major version 52 is newer than 51, the highest major version supported by this compiler.
It is recommended that the compiler be upgraded.

PWC6199: Generated servlet error:
type variable String is already defined in method <String,String><error>()

PWC6199: Generated servlet error:
cannot find symbol
symbol: class getTrailerFields
location: interface javax.servlet.http.HttpServletRequest

PWC6199: Generated servlet error:
cannot find symbol
symbol: class isTrailerFieldsReady
location: interface javax.servlet.http.HttpServletRequest

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class Supplier
location: interface javax.servlet.http.HttpServletResponse

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class setTrailerFields
location: interface javax.servlet.http.HttpServletResponse

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class getTrailerFields
location: interface javax.servlet.http.HttpServletResponse

PWC6199: Generated servlet error:
illegal start of type

PWC6199: Generated servlet error:
= expected

PWC6199: Generated servlet error:
';' expected

PWC6199: Generated servlet error:
<identifier> expected

PWC6199: Generated servlet error:
illegal start of type

PWC6199: Generated servlet error:
= expected

PWC6199: Generated servlet error:
';' expected

PWC6199: Generated servlet error:
<identifier> expected

PWC6199: Generated servlet error:
C:\Users\.m2\repository\org\jboss\spec\javax\servlet\jboss-servlet-api_4.0_spec\1.0.0.Final\jboss-servlet-api_4.0_spec-1.0.0.Final.jar(javax/servlet/ReadListener.class): major version 52 is newer than 51, the highest major version supported by this compiler.
It is recommended that the compiler be upgraded.

PWC6199: Generated servlet error:
cannot find symbol
symbol: class init
location: interface javax.servlet.Filter

PWC6199: Generated servlet error:
cannot find symbol
symbol: class destroy
location: interface javax.servlet.Filter

PWC6197: An error occurred at line: 129 in the jsp file: /front/main/jsp/view/../include/header.jspf
PWC6199: Generated servlet error:
package java.util.function does not exist

PWC6197: An error occurred at line: 18 in the jsp file: /front/main/jsp/view/../include/commonScript.jspf
PWC6199: Generated servlet error:
cannot find symbol
symbol: class Supplier
location: class javax.servlet.http.HttpServletResponseWrapper

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class Supplier
location: class javax.servlet.http.HttpServletResponseWrapper

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
interface methods cannot have body

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
interface methods cannot have body

PWC6199: Generated servlet error:
interface methods cannot have body

PWC6199: Generated servlet error:
interface methods cannot have body

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
interface methods cannot have body

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
interface methods cannot have body

PWC6199: Generated servlet error:
interface methods cannot have body

PWC6199: Generated servlet error:
interface methods cannot have body

PWC6199: Generated servlet error:
javax.servlet.http.HttpServletResponseWrapper is not abstract and does not override abstract method <error>() in javax.servlet.http.HttpServletResponse

PWC6197: An error occurred at line: 18 in the jsp file: /front/main/jsp/view/../include/commonScript.jspf
PWC6199: Generated servlet error:
method does not override or implement a method from a supertype

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: method getTrailerFields()
location: interface javax.servlet.http.HttpServletResponse

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
method does not override or implement a method from a supertype

PWC6199: Generated servlet error:
/domainPage_jsp.java uses unchecked or unsafe operations.

PWC6199: Generated servlet error:
Recompile with -Xlint:unchecked for details.

at org.apache.jasper.compiler.DefaultErrorHandler.javacError(DefaultErrorHandler.java:129)
at org.apache.jasper.compiler.ErrorDispatcher.javacError(ErrorDispatcher.java:299)
at org.apache.jasper.compiler.Compiler.generateClass(Compiler.java:392)
at org.apache.jasper.compiler.Compiler.compile(Compiler.java:453)
at org.apache.jasper.JspCompilationContext.compile(JspCompilationContext.java:625)
at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:374)
at org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:492)
at org.apache.jasper.servlet.JspServlet.service(JspServlet.java:378)
at javax.servlet.http.HttpServlet.service(HttpServlet.java:848)
at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:648)
at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:455)
at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:137)
at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:559)
at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:231)
at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1072)
at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:382)
at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:193)
at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1006)
at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:135)
at org.eclipse.jetty.server.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:255)
at org.eclipse.jetty.server.handler.HandlerCollection.handle(HandlerCollection.java:154)
at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:116)
at org.eclipse.jetty.server.Server.handle(Server.java:361)
at org.eclipse.jetty.server.AbstractHttpConnection.handleRequest(AbstractHttpConnection.java:485)
at org.eclipse.jetty.server.AbstractHttpConnection.headerComplete(AbstractHttpConnection.java:926)
at org.eclipse.jetty.server.AbstractHttpConnection$RequestHandler.headerComplete(AbstractHttpConnection.java:988)
at org.eclipse.jetty.http.HttpParser.parseNext(HttpParser.java:635)
at org.eclipse.jetty.http.HttpParser.parseAvailable(HttpParser.java:235)
at org.eclipse.jetty.server.AsyncHttpConnection.handle(AsyncHttpConnection.java:82)
at org.eclipse.jetty.io.nio.SelectChannelEndPoint.handle(SelectChannelEndPoint.java:627)
at org.eclipse.jetty.io.nio.SelectChannelEndPoint$1.run(SelectChannelEndPoint.java:51)
at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:608)
at org.eclipse.jetty.util.thread.QueuedThreadPool$3.run(QueuedThreadPool.java:543)
at java.lang.Thread.run(Thread.java:745)
16:25:07.796 [qtp1054507852-19 - /front/main/jsp/view/domainPage.jsp] WARN org.eclipse.jetty.servlet.ServletHandler - /front/main/jsp/view/domainPage.jsp
org.apache.jasper.JasperException: PWC6033: Error in Javac compilation for JSP

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
illegal start of type

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
= expected

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
';' expected

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
<identifier> expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
illegal start of type

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
= expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
';' expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
<identifier> expected

PWC6199: Generated servlet error:
illegal start of type

PWC6199: Generated servlet error:
= expected

PWC6199: Generated servlet error:
';' expected

PWC6199: Generated servlet error:
<identifier> expected

PWC6199: Generated servlet error:
illegal start of type

PWC6199: Generated servlet error:
= expected

PWC6199: Generated servlet error:
';' expected

PWC6199: Generated servlet error:
<identifier> expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
illegal start of type

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
= expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
';' expected

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
<identifier> expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
illegal start of type

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
= expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
';' expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
> expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
< expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
illegal start of expression

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
'(' expected

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
<identifier> expected

PWC6197: An error occurred at line: 129 in the jsp file: /front/main/jsp/view/../include/header.jspf
PWC6199: Generated servlet error:
package java.util.function does not exist

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class getHttpServletMapping
location: interface javax.servlet.http.HttpServletRequest

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class newPushBuilder
location: interface javax.servlet.http.HttpServletRequest

PWC6199: Generated servlet error:
C:\Users\.m2\repository\org\jboss\spec\javax\servlet\jboss-servlet-api_4.0_spec\1.0.0.Final\jboss-servlet-api_4.0_spec-1.0.0.Final.jar(javax/servlet/http/HttpUpgradeHandler.class): major version 52 is newer than 51, the highest major version supported by this compiler.
It is recommended that the compiler be upgraded.

PWC6199: Generated servlet error:
type variable String is already defined in method <String,String><error>()

PWC6199: Generated servlet error:
cannot find symbol
symbol: class getTrailerFields
location: interface javax.servlet.http.HttpServletRequest

PWC6199: Generated servlet error:
cannot find symbol
symbol: class isTrailerFieldsReady
location: interface javax.servlet.http.HttpServletRequest

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class Supplier
location: interface javax.servlet.http.HttpServletResponse

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class setTrailerFields
location: interface javax.servlet.http.HttpServletResponse

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class getTrailerFields
location: interface javax.servlet.http.HttpServletResponse

PWC6199: Generated servlet error:
illegal start of type

PWC6199: Generated servlet error:
= expected

PWC6199: Generated servlet error:
';' expected

PWC6199: Generated servlet error:
<identifier> expected

PWC6199: Generated servlet error:
illegal start of type

PWC6199: Generated servlet error:
= expected

PWC6199: Generated servlet error:
';' expected

PWC6199: Generated servlet error:
<identifier> expected

PWC6199: Generated servlet error:
cannot find symbol
symbol: class init
location: interface javax.servlet.Filter

PWC6199: Generated servlet error:
cannot find symbol
symbol: class destroy
location: interface javax.servlet.Filter

PWC6197: An error occurred at line: 129 in the jsp file: /front/main/jsp/view/../include/header.jspf
PWC6199: Generated servlet error:
package java.util.function does not exist

PWC6197: An error occurred at line: 18 in the jsp file: /front/main/jsp/view/../include/commonScript.jspf
PWC6199: Generated servlet error:
cannot find symbol
symbol: class Supplier
location: class javax.servlet.http.HttpServletResponseWrapper

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: class Supplier
location: class javax.servlet.http.HttpServletResponseWrapper

PWC6197: An error occurred at line: 9 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
interface methods cannot have body

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
interface methods cannot have body

PWC6199: Generated servlet error:
interface methods cannot have body

PWC6199: Generated servlet error:
interface methods cannot have body

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
interface methods cannot have body

PWC6197: An error occurred at line: 16 in the jsp file: /front/main/jsp/view/domainPage.jsp
PWC6199: Generated servlet error:
interface methods cannot have body

PWC6199: Generated servlet error:
interface methods cannot have body

PWC6199: Generated servlet error:
interface methods cannot have body

PWC6199: Generated servlet error:
javax.servlet.http.HttpServletResponseWrapper is not abstract and does not override abstract method <error>() in javax.servlet.http.HttpServletResponse

PWC6197: An error occurred at line: 18 in the jsp file: /front/main/jsp/view/../include/commonScript.jspf
PWC6199: Generated servlet error:
method does not override or implement a method from a supertype

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
cannot find symbol
symbol: method getTrailerFields()
location: interface javax.servlet.http.HttpServletResponse

PWC6197: An error occurred at line: 4 in the jsp file: /front/main/jsp/view/../include/../include/tmplUI.jsp
PWC6199: Generated servlet error:
method does not override or implement a method from a supertype

PWC6199: Generated servlet error:
/domainPage_jsp.java uses unchecked or unsafe operations.

PWC6199: Generated servlet error:
Recompile with -Xlint:unchecked for details.

at org.apache.jasper.compiler.DefaultErrorHandler.javacError(DefaultErrorHandler.java:129)
at org.apache.jasper.compiler.ErrorDispatcher.javacError(ErrorDispatcher.java:299)
at org.apache.jasper.compiler.Compiler.generateClass(Compiler.java:392)
at org.apache.jasper.compiler.Compiler.compile(Compiler.java:453)
at org.apache.jasper.JspCompilationContext.compile(JspCompilationContext.java:625)
at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:374)
at org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:492)
at org.apache.jasper.servlet.JspServlet.service(JspServlet.java:378)
at javax.servlet.http.HttpServlet.service(HttpServlet.java:848)
at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:648)
at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:455)
at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:137)
at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:559)
at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:231)
at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1072)
at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:382)
at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:193)
at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1006)
at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:135)
at org.eclipse.jetty.server.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:255)
at org.eclipse.jetty.server.handler.HandlerCollection.handle(HandlerCollection.java:154)
at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:116)
at org.eclipse.jetty.server.Server.handle(Server.java:361)
at org.eclipse.jetty.server.AbstractHttpConnection.handleRequest(AbstractHttpConnection.java:485)
at org.eclipse.jetty.server.AbstractHttpConnection.headerComplete(AbstractHttpConnection.java:926)
at org.eclipse.jetty.server.AbstractHttpConnection$RequestHandler.headerComplete(AbstractHttpConnection.java:988)
at org.eclipse.jetty.http.HttpParser.parseNext(HttpParser.java:635)
at org.eclipse.jetty.http.HttpParser.parseAvailable(HttpParser.java:235)
at org.eclipse.jetty.server.AsyncHttpConnection.handle(AsyncHttpConnection.java:82)
at org.eclipse.jetty.io.nio.SelectChannelEndPoint.handle(SelectChannelEndPoint.java:627)
at org.eclipse.jetty.io.nio.SelectChannelEndPoint$1.run(SelectChannelEndPoint.java:51)
at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:608)
at org.eclipse.jetty.util.thread.QueuedThreadPool$3.run(QueuedThreadPool.java:543)
at java.lang.Thread.run(Thread.java:745)
```

## cause

jetty버전을 올리고 최초 콘솔 로그인했을 때 똑같은 에러가 발생
glassfish에 버그가 많아서 9.3 밑으로 발생하는 오류라고 함.
mvn clean install해도 발생
[https://stackoverflow.com/questions/29005333/org-apache-jasper-jasperexception-while-accessing-jsps-in-jetty](https://stackoverflow.com/questions/29005333/org-apache-jasper-jasperexception-while-accessing-jsps-in-jetty)

## solved

jetty 올릴 때, argument로 -Dorg.apache.jasper.compiler.disablejsr199=true 를 추가한다