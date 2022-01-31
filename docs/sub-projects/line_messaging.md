---
layout: post
title: Line messaging API 적용 노트
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Sub Projects
has_children: false
nav_order: 7
nav_exclude: true
---

# Line messaging API 적용 노트

1. webhook 설정 - ssl설정

1. controller작성 

기존 프로젝트에 line 메시징 API만 적용해야하는 환경이라, 라인sdk에서 적용한 스프링버전보다 높은 상황

자바는 컴파일은 1.8,

런타임은 12로 되는 상황

간단한 컨트롤러 작성

```java
@Slf4j
@RestController
@LineMessageHandler
public class LineMessageController {
    @EventMapping
    public void handleTextMessageEvent(MessageEvent<TextMessageContent> event) {
        log.info("event: " + event);
        final String originalMessageText = event.getMessage().getText();
        //return new TextMessage(originalMessageText);
    }

    @EventMapping
    public void handleDefaultMessageEvent(Event event) {
        System.out.println("event: " + event);
    }
}
```

에러1) 

```
14:34:33.586 010-exec-6 ERROR ineMessageHandlerSupport:217 dispatch        Unsupported event type. MessageEvent(replyToken=3406d4539061400ca6d199bc2bad16e6, source=UserSource(userId=Uec5b4b45de71961100290f7a7ff61d0a), message=TextMessageContent(id=15206625437201, text=123, emojis=null, mention=null), timestamp=2021-12-07T05:34:32.441Z, mode=ACTIVE)
java.lang.UnsupportedOperationException: Unsupported event type. MessageEvent(replyToken=3406d4539061400ca6d199bc2bad16e6, source=UserSource(userId=Uec5b4b45de71961100290f7a7ff61d0a), message=TextMessageContent(id=15206625437201, text=123, emojis=null, mention=null), timestamp=2021-12-07T05:34:32.441Z, mode=ACTIVE)
	at com.linecorp.bot.spring.boot.support.LineMessageHandlerSupport.lambda$dispatchInternal$6(LineMessageHandlerSupport.java:226)
	at java.util.Optional.orElseThrow(Optional.java:290)
	at com.linecorp.bot.spring.boot.support.LineMessageHandlerSupport.dispatchInternal(LineMessageHandlerSupport.java:226)
	at com.linecorp.bot.spring.boot.support.LineMessageHandlerSupport.dispatch(LineMessageHandlerSupport.java:213)
	at com.linecorp.bot.spring.boot.support.LineMessageHandlerSupport.lambda$callback$4(LineMessageHandlerSupport.java:206)
	at java.util.ArrayList.forEach(ArrayList.java:1259)
	at com.linecorp.bot.spring.boot.support.LineMessageHandlerSupport.callback(LineMessageHandlerSupport.java:205)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:190)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:138)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:105)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:878)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:792)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1040)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:943)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:665)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:750)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:227)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at brave.servlet.TracingFilter.doFilter(TracingFilter.java:68)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at brave.servlet.TracingFilter.doFilter(TracingFilter.java:87)
	at org.springframework.cloud.sleuth.instrument.web.LazyTracingFilter.doFilter(TraceWebServletAutoConfiguration.java:141)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:97)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97)
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:542)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:143)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:357)
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:374)
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:893)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1707)
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:748)
```

분석1) springboot로 띄울때, 디버깅해보면, LineMessageHandler를 선언한 클래스는 스프링에 등록되어 있지만, '@EventMapping을 선언한 메소드는 찾지 못하는 문제 발생

시도1) @SpringBootApplication(scanBasePackages = "~~~~")가 기존 프로젝트에 추가된 상황이였는데, scan대상이 잘못되고 있는 것인가 해서, scanBasePackages를 제거하고 올리면 성공

하지만  scanBasePackages 설정은 다른 컨테이너와의 상속문제로 반드시 필요한 설정이라 뺄 수가 없음

시도2) 라인sdk의 echo를 다시 유심히 봤다. 차이는 @RestController뿐. @RestController를 제거해보니 성공

정리는 간단했지만, spring 버전차이를 중심으로 디버깅하다보니 삽질이 길었다.

문제는 @RestController 중복선언으로 보인다.

LineMessageHandlerSupport.java내용을 보면 거기에도 @RestController를 자체적으로 선언하고 있다.

```java
@Slf4j
@RestController
@Import(ReplyByReturnValueConsumer.Factory.class)
@ConditionalOnProperty(name = "line.bot.handler.enabled", havingValue = "true", matchIfMissing = true)
public class LineMessageHandlerSupport {
```

해당 파일의 주석으로도 되어있다. class에는 @LineMessageHandler를 method에는 @EventMappinig을 선언하라고...