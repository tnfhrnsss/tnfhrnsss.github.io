---
layout: post
title: Executing JavaScript with Nashorn in Java 15+ Replacing jjs
date: 2025-02-23 14:21:00
last_modified_at : 2025-02-23 14:21:00
parent: Java
grand_parent: Language
has_children: false
nav_exclude: true
tags: [jjs, nashorn]
---


# 요구사항

- 기존에는 jjs를 통해서 리눅스 서버에서 Javascript 파일을 실행시켜왔는데, java 11에는 jjs가 deprecated되고 java15 이후부터 jjs가 삭제되어서 사용할 수 없기 때문에 대체 방식을 고민해야했다.

# jjs 대체 기술 검토

## 1. 검토 방향

- java8부터 17+까지 호환가능한 방식이여야한다.
- 설치한 서버에서 컴파일없이 수정이 가능해야한다.

## 2. 방식

- 1안) java의 jjs가 아닌, jjs 라이브러리 파일을 위치해놓고 별도 주입
    - 단점:
        - Java 8의 Nashorn 엔진은 Java 15 이후의 최신 JVM과 완전히 호환되지 않을 수 있음.
- 2안) GraalVM
    - 단점
        - GraalVM (21.x 이상)은 **Java 8을 직접 지원하지 않음**.
        - 기존 프로젝트와 통합이 어려움
- 3안) Mozilla Rhino (JavaScript 엔진)
    - 단점
        - Rhino보다는 Nashorn사용 권장(속도 측면 비추)
- 4안) javascript를 통해 실행시키는 방법자체를 변경
    - jjs의존하지 않는 방식으로 논의 → curl로 실행하는 쉘 스크립트 작성 등
    - 단점
        - 유지보수 어려움
- 5안) org.openjdk.nashorn 사용 (택) - 상세설명은 하단에서..
- 논외
    - python, HttpURLConnection, nodejs ..

# openjdk.nashorn 적용

## 1. 정리

- Nashorn이란?
    - Nashorn은 JDK 8에서 처음 도입되었고,
    - JDK 11에서 Deprecated(사용 중단 예정) 되었으며,
    - JDK 15에서 완전히 제거되었다.
- openjdk.nashorn은
    - 15 이후부터는 깃헙 저장소에서 GPLv2 with Classpath Exception 라이선스로 유지됨(**JAR 형태로 포함하여 상용 프로젝트에서 사용 가능**.)
        - [https://github.com/openjdk/nashorn](https://github.com/openjdk/nashorn){:target="_blank"}
        - [https://openjdk.org/projects/nashorn/](https://openjdk.org/projects/nashorn/){:target="_blank"}
    - 자바버전별 호환
        - nashorn 15.2~15.6의 경우org/openjdk/nashorn/api/scripting/NashornScriptEngineFactory 최소 버전이 java11이여서 8에서는 실행시킬수 없다.
        - 15.1, 15.1.1은 최소 java15이상만 가능 (특이하다.)
        - 15.0은 8 가능
- 단점
    - OpenJDK Nashorn 프로젝트는 현재 활발히 유지보수되지 않음.

### 2.  설계

- 깃헙으로 오면서(15이후), 독립실행형 (Main)클래스가 제거되었다.
    - 그래서 openjdk-nashorn의 ScriptEngineManager을 통해 실행시키는 코드 구현 필요하다.
- openjdk-nashorn 버전별 자바와의 호환 차이

    | **openjdk.nashorn 버전**  | **런타임 최소버전**  |
    |--------------|--------------|
    | 15.1, 15.1.1  | java15            |
    | 15.2~15.6(latest)   | java11            |
    | 15.0   | java8            |


    - 자바15이후부턴 security privilege하는 부분이 문제가 있어서 15.0사용 불가
    - 자바15이후 지원을 위해 15.6으로 하면, 사용가능하지만
        - 반대로 자바 낮은 버전에서 사용못함
- 자바8에서는 jjs가 있어서 잘되지만 자바17에서 하면 "engine" is null 오류
    
    ```java
    Error: No script nashorn engine found
    java.lang.NullPointerException: Cannot invoke 
    "javax.script.ScriptEngine.eval(java.io.Reader)" because "engine" is null
    at NashornExecutor.main(NashornExecutor.java:26)
    ```
    
- nashorn 15.0은 java17에서 nashorn모듈네임("engine" is null 오류)을 찾을수가 없고, 그게 java 15부터 deprecated되어서 15.1부터는 dependency를 걸면 java8에서 unsupported발생한다.
- 결론은
    - 15미만까지는 기존처럼 java의 jjs를 사용하고
    - java15+의 경우는 Main메서드를 통해 openjdk-nashorn engine을 통해 실행하도록 변경
        - 방식은 외부 라이브러리 참조방식과 내부 의존성 추가 방식

## 3. 구현

### 1안) 라이브러리 외부 참조방식

- **필요한 라이브러리**
    - `nashorn-core-15.x.jar`
    - `asm-9.5.jar`, `asm-util-9.5.jar`
- Main메서드가 실행될 때 리플렉션을 통해 외부 라이브러리 참조하도록 함
    
    ```java
    mport java.io.File;
    import java.io.FileReader;
    import javax.script.ScriptEngine;
    import javax.script.ScriptEngineManager;
    
    public class NashornExecutor {
        public static void main(String[] args) {
              try {
                Class<?> clazz = Class.forName("org.openjdk.nashorn.api.scripting.NashornScriptEngineFactory");
                ScriptEngineFactory factory = (ScriptEngineFactory) clazz.getDeclaredConstructor().newInstance();
    
                engine = factory.getScriptEngine();
                System.out.println("ScriptEngine created: " + engine);
    
                // JavaScript 코드 실행
    
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }
    
    ```
    

- 실행
    
    ```
    cmd) java -cp "nashorn-core-15.0.jar:asm-9.5.jar:asm-util-9.5.jar" \
    NashornExecutor test.js 
    ```
    

### 2안) 내부 maven 의존성 추가방식

- pom.xml에 의존성 추가
    
    ```xml
     <dependency>
        <groupId>org.openjdk.nashorn</groupId>
        <artifactId>nashorn-core</artifactId>
        <version>15.6</version>
    </dependency>
    ```
    

- Main메서드를 통해 nashorn엔진 실행 코드 작성
    
    ```java
    import java.io.File;
    import java.io.FileReader;
    import javax.script.ScriptEngine;
    import javax.script.ScriptEngineManager;
    
    public class NashornExecutor {
        public static void main(String[] args) {
            ScriptEngineManager manager = new ScriptEngineManager();
            ScriptEngine engine = manager.getEngineByName("nashorn");
            if (engine == null) {
                System.out.println("Error: nashorn engine not found.");
            }
    
            try {
                engine.eval(new FileReader(new File("javascript file")));
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    ```
    

- 실행
    - java8 ~ java15 미만 버전
    
    ```
    cmd> jjs -cp {MAVEN_TARGET}\nashornexecutor-1.0.0-SNAPSHOT.jar test.js
    
    ```
    
    - java15 이후 버전
    
    ```java
    cmd> java -cp {MAVEN_TARGET}\nashornexecutor-1.0.0-SNAPSHOT.jar \
     NashornExecutor test.js
    ```