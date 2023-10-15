---
layout: post
title: BeanDefinitionOverrideException
description: BeanDefinitionOverrideException
date: 2023-10-15 20:53:00
last_modified_at : 2023-10-15 20:53:00
parent: Errors
has_children: false
nav_exclude: true
---

# error log

```
java.lang.IllegalStateException: Failed to load ApplicationContext
Caused by: 
org.springframework.beans.factory.support.BeanDefinitionOverrideException: 
Invalid bean definition with name 'invalidMaxFileSizeException' 
defined in class path resource [spectra/attic/coreasset/depot/exception/rest/DepotExceptionConfig.class]: 
Cannot register bean definition [Root bean: class [null]; scope=; abstract=false; lazyInit=null; autowireMode=3; 
dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=depotExceptionConfig; 
factoryMethodName=invalidMaxFileSizeException; initMethodName=null; destroyMethodName=(inferred); 
defined in class path resource [spectra/attic/coreasset/depot/exception/rest/DepotExceptionConfig.class]] 
for bean 'invalidMaxFileSizeException': There is already [Root bean: class [null]; 
scope=; abstract=false; lazyInit=null; autowireMode=3; dependencyCheck=0; 
autowireCandidate=true; primary=false; factoryBeanName=fileExceptionConfig; f
actoryMethodName=invalidMaxFileSizeException; initMethodName=null; 
destroyMethodName=(inferred); 
defined in class path resource 
[spectra/attic/talk/messengerconnector/thirdparty/file/exception/rest/FileExceptionConfig.class]] bound.
```

# cause

- spring boot 시작하면서 빈을 등록할 때, 동일이름으로 여러 빈이 정의되어 있어서 발생하는 에러

# problem

- 발생 배경에는 제품 전반적으로 모듈 구조가 변경되면서 dependency관계도 재정의가 되었다. 그래서 기존에는 참조하고 있지 않았던 다른 어플리케이션의 Exception 모델이 빈으로 등록되면서 내부적으로 갖고 있던 동일 빈과 충돌이 발생하게 된 것이다.

```java
@Bean
public ExceptionModel minFileSizeException() {
  return new ExceptionModel(MinFileSizeException.class, HttpStatus.BAD_REQUEST, FileRestErrorCode.INVALID_MIN_FILE_SIZE_ERROR);
}
```

# solved

- 1안) 빈 이름 변경
- 2안) @Primary를 써서 우선순위를 설정하는 방법
- 결국 1안으로 진행했다.

## AnnotationBeanNameGenerator을 고려한다면?

- 중복된 빈 이름을 막기 위해 AnnotationBeanNameGenerator을 사용할 수 있다.
- 결론은 이 케이스는 @Bean으로 등록한 것이라 적용이 불가능하다.
- @Bean으로 설정한 메소드는 @ComponentScan 및 AnnotationBeanNameGenerator와 함께 사용되는 것이 아니라 별도의 방식으로 사용되기 때문에 그러하다. 즉 스프링 컨테이너는 @Bean으로 등록된 구성 클래스는 스캔하지 않는다.