---
layout: post
title: store-jpa 모듈을 만들고 나니 Not a managed type 에러 날때
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
tags: [jpa, springboot]
---

#### error log

```
11:30:46.596 main WARN stractApplicationContext:557 refresh Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: 

Error creating bean with name 'aaaService' defined in file []: 
Unsatisfied dependency expressed through constructor parameter 0; nested exception is 
at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:769) 
at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:218) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireConstructor(AbstractAutowireCapableBeanFactory.java:1341) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1187) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:555) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:515) 
at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) 
at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) 
at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:845) 
at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:877) 
at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:549) at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:140) 
at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:742) 
at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:389) 
at org.springframework.boot.SpringApplication.run(SpringApplication.java:311) 
at org.springframework.boot.SpringApplication.run(SpringApplication.java:1213) 
at org.springframework.boot.SpringApplication.run(SpringApplication.java:1202) 
at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:769) 
at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:218) at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireConstructor(AbstractAutowireCapableBeanFactory.java:1341) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1187) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:555) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:515) 
at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) 
at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:277) at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1251) 
at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1171) at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:857) 
at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:760) 
... 19 common frames omitted

Caused by: org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'internalGatewayRepository': Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: 
Not a managed type: class AAAJpo at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1778) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:593) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:515) 
at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) 
at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) 
at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:277) 
at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1251) 
at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1171) 
at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:857) 
at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:760)... 33 common frames omitted

Caused by: java.lang.IllegalArgumentException: Not a managed type: class AAAGatewayJpo

at org.hibernate.metamodel.internal.MetamodelImpl.managedType(MetamodelImpl.java:552) 
at org.springframework.data.jpa.repository.support.JpaMetamodelEntityInformation.<init>(JpaMetamodelEntityInformation.java:74) 
at org.springframework.data.jpa.repository.support.JpaEntityInformationSupport.getEntityInformation(JpaEntityInformationSupport.java:66) 
at org.springframework.data.jpa.repository.support.JpaRepositoryFactory.getEntityInformation(JpaRepositoryFactory.java:201) 
at org.springframework.data.jpa.repository.support.JpaRepositoryFactory.getTargetRepository(JpaRepositoryFactory.java:151) 
at org.springframework.data.jpa.repository.support.JpaRepositoryFactory.getTargetRepository(JpaRepositoryFactory.java:134) 
at org.springframework.data.jpa.repository.support.JpaRepositoryFactory.getTargetRepository(JpaRepositoryFactory.java:65) 
at org.springframework.data.repository.core.support.RepositoryFactorySupport.getRepository(RepositoryFactorySupport.java:305) at org.springframework.data.repository.core.support.RepositoryFactoryBeanSupport.lambda$afterPropertiesSet$5(RepositoryFactoryBeanSupport.java:297) at org.springframework.data.util.Lazy.getNullable(Lazy.java:211) 
at org.springframework.data.util.Lazy.get(Lazy.java:94) 
at org.springframework.data.repository.core.support.RepositoryFactoryBeanSupport.afterPropertiesSet(RepositoryFactoryBeanSupport.java:300) 
at org.springframework.data.jpa.repository.support.JpaRepositoryFactoryBean.afterPropertiesSet(JpaRepositoryFactoryBean.java:121) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1837) 
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1774) 
... 44 common frames omitted

Process finished with exit code 1
```


#### cause

SpringApplication으로 서비스 올릴 때, scan하는 패키지는 test.www인데 springboot로 올리는 application 파일은 test.www.aaa.boot아래에 있었다. 
이걸 test.www.aaa 하위로 옮기고 나니 해결됨