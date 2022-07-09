---
layout: post
title: The write format 1 is smaller than the supported format 2
date: 2022-07-09 21:32:00
last_modified_at : 2022-07-09 21:32:00
parent: Errors
has_children: false
nav_exclude: true
tags: [hystrix, msa, feign]
---

# The write format 1 is smaller than the supported format 2 [2.0.206/5]

## error

```
org.h2.jdbc.JdbcSQLNonTransientException: General error: "The write format 1 is smaller than the supported format 2 [2.0.206/5]" [50000-206]
at org.h2.message.DbException.getJdbcSQLException(DbException.java:573) ~[h2-2.0.206.jar:2.0.206]
at org.h2.message.DbException.getJdbcSQLException(DbException.java:496) ~[h2-2.0.206.jar:2.0.206]
at org.h2.message.DbException.get(DbException.java:216) ~[h2-2.0.206.jar:2.0.206]
at org.h2.mvstore.db.Store.convertMVStoreException(Store.java:166) ~[h2-2.0.206.jar:2.0.206]
at org.h2.mvstore.db.Store.<init>(Store.java:140) ~[h2-2.0.206.jar:2.0.206]
at org.h2.engine.Database.<init>(Database.java:324) ~[h2-2.0.206.jar:2.0.206]
at org.h2.engine.Engine.openSession(Engine.java:92) ~[h2-2.0.206.jar:2.0.206]
at org.h2.engine.Engine.openSession(Engine.java:222) ~[h2-2.0.206.jar:2.0.206]
at org.h2.engine.Engine.createSession(Engine.java:201) ~[h2-2.0.206.jar:2.0.206]
at org.h2.engine.SessionRemote.connectEmbeddedOrServer(SessionRemote.java:338) ~[h2-2.0.206.jar:2.0.206]
at org.h2.jdbc.JdbcConnection.<init>(JdbcConnection.java:117) ~[h2-2.0.206.jar:2.0.206]
at org.h2.Driver.connect(Driver.java:59) ~[h2-2.0.206.jar:2.0.206]
at com.zaxxer.hikari.util.DriverDataSource.getConnection(DriverDataSource.java:138) ~[HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.pool.PoolBase.newConnection(PoolBase.java:358) ~[HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.pool.PoolBase.newPoolEntry(PoolBase.java:206) ~[HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.pool.HikariPool.createPoolEntry(HikariPool.java:477) [HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.pool.HikariPool.checkFailFast(HikariPool.java:560) [HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.pool.HikariPool.<init>(HikariPool.java:115) [HikariCP-3.4.5.jar:na]
at com.zaxxer.hikari.HikariDataSource.getConnection(HikariDataSource.java:112) [HikariCP-3.4.5.jar:na]
at org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration.lambda$h2Console$0(H2ConsoleAutoConfiguration.java:72) [spring-boot-autoconfigure-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.beans.factory.ObjectProvider.ifAvailable(ObjectProvider.java:93) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration.h2Console(H2ConsoleAutoConfiguration.java:71) [spring-boot-autoconfigure-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_282]
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_282]
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_282]
at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_282]
at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:154) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:652) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:637) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1341) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1181) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:556) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:516) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:324) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:322) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:207) ~[spring-beans-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.boot.web.servlet.ServletContextInitializerBeans.getOrderedBeansOfType(ServletContextInitializerBeans.java:211) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.servlet.ServletContextInitializerBeans.getOrderedBeansOfType(ServletContextInitializerBeans.java:202) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.servlet.ServletContextInitializerBeans.addServletContextInitializerBeans(ServletContextInitializerBeans.java:96) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.servlet.ServletContextInitializerBeans.<init>(ServletContextInitializerBeans.java:85) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.getServletContextInitializerBeans(ServletWebServerApplicationContext.java:255) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.selfInitialize(ServletWebServerApplicationContext.java:229) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.embedded.tomcat.TomcatStarter.onStartup(TomcatStarter.java:53) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5161) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1384) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1374) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at java.util.concurrent.FutureTask.run(FutureTask.java:266) ~[na:1.8.0_282]
at org.apache.tomcat.util.threads.InlineExecutorService.execute(InlineExecutorService.java:75) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:134) ~[na:1.8.0_282]
at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:909) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.core.StandardHost.startInternal(StandardHost.java:829) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1384) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1374) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at java.util.concurrent.FutureTask.run(FutureTask.java:266) ~[na:1.8.0_282]
at org.apache.tomcat.util.threads.InlineExecutorService.execute(InlineExecutorService.java:75) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:134) ~[na:1.8.0_282]
at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:909) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.core.StandardEngine.startInternal(StandardEngine.java:262) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.core.StandardService.startInternal(StandardService.java:433) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.core.StandardServer.startInternal(StandardServer.java:930) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.apache.catalina.startup.Tomcat.start(Tomcat.java:486) ~[tomcat-embed-core-9.0.46.jar:9.0.46]
at org.springframework.boot.web.embedded.tomcat.TomcatWebServer.initialize(TomcatWebServer.java:123) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.embedded.tomcat.TomcatWebServer.<init>(TomcatWebServer.java:104) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory.getTomcatWebServer(TomcatServletWebServerFactory.java:440) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory.getWebServer(TomcatServletWebServerFactory.java:193) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.createWebServer(ServletWebServerApplicationContext.java:178) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.onRefresh(ServletWebServerApplicationContext.java:158) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:545) ~[spring-context-5.2.15.RELEASE.jar:5.2.15.RELEASE]
at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:143) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:755) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:747) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:402) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.SpringApplication.run(SpringApplication.java:312) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.SpringApplication.run(SpringApplication.java:1247) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at org.springframework.boot.SpringApplication.run(SpringApplication.java:1236) ~[spring-boot-2.3.12.RELEASE.jar:2.3.12.RELEASE]
at spectra.attic.coreasset.ecosystem.registry.RegistryApplication.main(RegistryApplication.java:16) ~[classes/:na]
Caused by: org.h2.mvstore.MVStoreException: The write format 1 is smaller than the supported format 2 [2.0.206/5]
at org.h2.mvstore.DataUtils.newMVStoreException(DataUtils.java:1004) ~[h2-2.0.206.jar:2.0.206]
at org.h2.mvstore.MVStore.getUnsupportedWriteFormatException(MVStore.java:1059) ~[h2-2.0.206.jar:2.0.206]
at org.h2.mvstore.MVStore.readStoreHeader(MVStore.java:878) ~[h2-2.0.206.jar:2.0.206]
at org.h2.mvstore.MVStore.<init>(MVStore.java:455) ~[h2-2.0.206.jar:2.0.206]
at org.h2.mvstore.MVStore$Builder.open(MVStore.java:4052) ~[h2-2.0.206.jar:2.0.206]
at org.h2.mvstore.db.Store.<init>(Store.java:129) ~[h2-2.0.206.jar:2.0.206]
... 77 common frames omitted
```

## solve

h2버전 업 전에 생성했던 h2파일이 이전 버전이어서 생긴 문제로 보여서

이전 h2파일 삭제하고 다시 재기동!