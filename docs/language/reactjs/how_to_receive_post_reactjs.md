---
layout: post
title: How to receive data from a post request in reactjs
description: How to receive data from a post request in reactjs
date: 2024-12-30 15:21:00
last_modified_at : 2024-12-30 15:21:00
parent: Reactjs
grand_parent: Language
has_children: false
nav_exclude: true
---

# Problem

- 전달 받을 파라미터가 많다.
- 파라미터 값에는 한글이 포함된 데이터도 있다. (get일 경우 인코딩 및 암호화 적용이 필수)
- post로 호출가능한 URL제공을 요청받았다..
- reactjs로는 post요청을 받을 수 없다.
- 프로젝트는 reactjs로 frontend를 담당하며 spring boot를 통해 서비스하고 있다.

# Try 1. jar → jsp & war

## 방법

- post요청을 받기 위한 path mapping 선언
- post요청을 받은 후 인코딩 및 암호화 등 필요한 전처리 작업 진행한다.
- request 내용 그대로 jsp로 redirect한다.
- jsp에서 브라우저 session storage를 통해 파라미터를 저장하고 동일 path에 대해 get으로 호출한다.
- jar로 배포하던 것을 war로 변경한다.

## 작업내용

- pom.xml
    
    ```prolog
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns="http://maven.apache.org/POM/4.0.0"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <packaging>war</packaging>
    
        <dependencies>
           .....skip.....
            <dependency>
                <groupId>org.apache.tomcat.embed</groupId>
                <artifactId>tomcat-embed-jasper</artifactId>
            </dependency>
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>jstl</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
            </dependency>
        </dependencies>
    
        <build>
            <finalName>${project.artifactId}-boot-${project.version}</finalName>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                     <configuration>
                        <mainClass>com.test.Application</mainClass>
                        <layout>WAR</layout>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </project>
    
    ```
    
- Controller.java
    
    ```prolog
    @PostMapping("/**/{path:[^\\.]*}")
    public void postForward(
          @PathVariable String path,
          HttpServletRequest request,
          HttpServletResponse response,
          HttpEntity<String> httpEntity
    ) {
      String forwardUrl = UriMaker.makeUriWithQuery(path, request, httpEntity);
      response.setHeader("Location", forwardUrl);
      response.setStatus(302);
    }
    ```
    
- WEB-INF/post.jsp
    
    ```prolog
    <%@ page import="java.util.Enumeration" %>
    <%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
    
    <%
        request.setCharacterEncoding("UTF-8");
    
        Enumeration<String> params = request.getParameterNames();
        StringBuilder paramString = new StringBuilder();
    
        while (params.hasMoreElements()) {
            String name = params.nextElement();
            paramString.append(name);
            paramString.append(request.getParameter(name));
    
            if (params.hasMoreElements()) {
                paramString.append("_");
            }
        }
    %>
    <!DOCTYPE html>
    <html lang="ko">
    <body>
    <script type="text/javascript">
        const setStorage = (key, value) => {
            if (typeof value === 'object') {
                sessionStorage.setItem(key, JSON.stringify(value));
            } else {
                sessionStorage.setItem(key, value);
            }
        };
    </script>
    <script type="text/javascript">
        const CONTEXT_PATH = location.pathname;
        const params = '<%= paramString %>'
        const jsonParams = {};
    
        params.split(PARAM_SEPARATOR).forEach(param => {
            const [key, value] = param.split("_");
            jsonParams[key] = value;
        });
        setStorage("keys", jsonParams);
    
        const url = jsonParams.url ? jsonParams.url: location.origin + CONTEXT_PATH;
        location.href = url
    </script>
    </body>
    </html>
    ```
    

# Try 2. Thymeleaf

## 방향

- html페이지에서 필요한 처리를 진행가능
- jar로 배포
- post로 호출되는 것은 모두 post.html로 포워딩한다.

## 작업내용

- pom.xml
    
    ```prolog
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns="http://maven.apache.org/POM/4.0.0"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
        <dependencies>
           .....skip.....
           
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-thymeleaf</artifactId>
            </dependency>
        </dependencies>
    
        <build>
            <finalName>${project.artifactId}-boot-${project.version}</finalName>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    </project>
    
    ```
    
- Controller.java
    
    ```prolog
    @PostMapping("/**/{path:[^\\.]*}")
    public String handlePost(HttpServletRequest request, Model model) {
        model.addAttribute("paramString", UriMaker.makeUriWithQuery(request));
        return "post";
    }
    ```
    
- resources/templates 폴더 생성
- src/main/resources/templates/post.html
    - 자바스크립트 인라인을 사용해야한다.
        - [https://minisiri.tistory.com/43](https://minisiri.tistory.com/43){:target="_blank"}
    
    ```prolog
    <!DOCTYPE html>
    <html lang="ko" xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>CS Talk - Guest</title>
        <script type="text/javascript">
            const setStorage = (key, value) => {
                if (typeof value === 'object') {
                    sessionStorage.setItem(key, JSON.stringify(value));
                } else {
                    sessionStorage.setItem(key, value);
                }
            };
        </script>
    </head>
    <body>
    <script th:inline="javascript">
        /*<![CDATA[*/
        const CONTEXT_PATH = location.pathname;
        const params = /*[[${paramString}]]*/ '';
        const jsonParams = {};
    
        try {
            params.split(PARAM_SEPARATOR).forEach(param => {
                const [key, value] = param.split("_");
                jsonParams[key] = value;
            });
        } catch (e) {
            console.log(e);
        }
    
        setStorage("keys", jsonParams);
    
        const url = jsonParams.url ? jsonParams.url : location.origin + CONTEXT_PATH;
        window.location.href = url;
        /*]]>*/
    </script>
    </body>
    </html>
    
    ```
    
- application.yml
    - 필수 아님
    
    ```prolog
    spring:
      thymeleaf:
        prefix: classpath:/templates/
        suffix: .html

    ```
    
    - ThymeleafViewResolver는 뷰 이름을 접두사와 접미사로 둘러싸 리소스를 찾는다.
    - 접두어와 접미어의 기본값은 각각 'classpath:/templates/'와 '.html' 이다. 변경이 필요한 경우 application.yml에 위의 설정을 추가
    - ThymeleafViewResolver동일한 이름의 bean을 제공하여 재정의할 수 있다.
    - classpath:/templates/ 경로에서 .html 확장자의 템플릿 파일을 찾습니다.
    - Spring Boot에서 기본적으로 활성화된다.

# 결론

- jsp&war를 적용했다가, 동료 리뷰를 듣고 다시 타임리프로 변경 및 적용했다.
- 추가적으로 에러페이지 핸들링도 추가했다.
- 이유는❓
    - war로 배포 argocd 및 개발자 로컬 환경 셋팅을 다시 해야하는 문제가 있었다. 😭
    - 유지보수 측면에서도 타임리프가 편할 것이라고 판단했다.(배경을 몰라도 프론트엔드 유지보수 작업에 영향이 적은 편이 중요도 측면에서 우선했다.)

# Reference

- [https://lahezy.tistory.com/104](https://lahezy.tistory.com/104){:target="_blank"}
- [https://www.baeldung.com/spring-circular-view-path-error](https://www.baeldung.com/spring-circular-view-path-error){:target="_blank"}