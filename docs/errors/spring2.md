---
layout: post
title: PathVariable annotation was empty on param 0.
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Errors
has_children: false
nav_exclude: true
tags: [spring, springboot]
---

## cause and solution
@PathVariable("id") 일케 해야하는데, 그냥 @PathVariable이것만 선언해서 발생

## error log

```
org.springframework.beans.factory.UnsatisfiedDependencyException: 
Error creating bean with name 'aaa' defined in file [aaa.class]: 
Unsatisfied dependency expressed through constructor parameter 0; 
nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: 
Error creating bean with name 'bbb' defined in file [bbb.class]:
Unsatisfied dependency expressed through constructor parameter 9; 
nested exception is org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'CCCClient': FactoryBean threw exception on object creation; 
nested exception is java.lang.IllegalStateException: **PathVariable annotation was empty on param 0**.

... 19 common frames omitted

Caused by: org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'CCCOperatorClient': 
FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: 
PathVariable annotation was empty on param 0.

at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:857)
at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:760)
... 33 common frames omitted

Caused by: java.lang.IllegalStateException: PathVariable annotation was empty on param 0.
at feign.Util.checkState(Util.java:130)
at org.springframework.cloud.openfeign.annotation.PathVariableParameterProcessor.processArgument(PathVariableParameterProcessor.java:52)
at org.springframework.cloud.openfeign.support.SpringMvcContract.processAnnotationsOnParameter(SpringMvcContract.java:292)
at feign.Contract$BaseContract.parseAndValidateMetadata(Contract.java:110)(FactoryBeanRegistrySupport.java:171)

... 45 common frames omitted
```