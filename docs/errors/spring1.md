---
layout: post
title: Failed to instantiate [javax.servlet.Filter]...
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
---

## Failed to instantiate [javax.servlet.Filter]: Factory method 'springSecurityFilterChain' threw exception; nested exception is java.lang.NullPointerException 에러


## problems
springboot로 application 올리다가 에러 발생

## error log


```
07:54:50.971 main WARN stractApplicationContext:557 refresh **Exception encountered during context initialization 
- cancelling refresh attempt: org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'springSecurityFilterChain' defined in class path resource** 
[org/springframework/security/config/annotation/web/configuration/WebSecurityConfiguration.class]: 
Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: 
Failed to instantiate [javax.servlet.Filter]: Factory method 'springSecurityFilterChain' threw exception; nested exception is java.lang.NullPointerException

07:54:54.008 main WARN o.a.j.l.DirectJDKLog:173 log The web application [ROOT] appears to have started a thread named [RxIoScheduler-1 (Evictor)] but has failed to stop it. 
This is very likely to create a memory leak. Stack trace of thread:

07:54:54.037 main ERROR o.s.b.SpringApplication :821 reportFailureApplication run failed
org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'springSecurityFilterChain' defined in class path resource 
[org/springframework/security/config/annotation/web/configuration/WebSecurityConfiguration.class]: 
Bean instantiation via factory method failed; 
nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [javax.servlet.Filter]: 
Factory method 'springSecurityFilterChain' threw exception; nested exception is java.lang.NullPointerException
at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:627)
at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:456)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1321)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1160)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:555)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:515)
at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320)
at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222)
at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318)
at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)
at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:307)
at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)
at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:845)
at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:877)
at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:549)
at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:140)
at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:742)
at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:389)
at org.springframework.boot.SpringApplication.run(SpringApplication.java:311)
at org.springframework.boot.SpringApplication.run(SpringApplication.java:1213)
at org.springframework.boot.SpringApplication.run(SpringApplication.java:1202)
at xxx.XXXApplication.main(XXXApplication.java:18)

Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [javax.servlet.Filter]: 
Factory method 'springSecurityFilterChain' threw exception; nested exception is java.lang.NullPointerException
at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:185)
at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:622)
... 21 common frames omitted

```

## cause
에러의 npe부분을 보면 springboot올릴때, WebSecurityConfig를 읽어가도록 했는데. WebSecurityConfig에서 읽어갈 설정이 yml에 없어서 널포인트 나는 것....
when some property of yml file doesn't exist, then occurs this error.

## solved
본인의 경우는 profile을 달리했는데, 그걸 서비스 올릴때 지정안해줘서 디폴트 yml보고 올라가다가 해당 security설정이 없어서 그런거라...profile을 지정해주면 된다.

In my case, a variable of WebSecurityConfig.class was a property of yml file. 
But I didn't assign the right profile(dev, local...) of spring boot application.
So when WebSecurityConfig was loaded, it cannot find value of property.
