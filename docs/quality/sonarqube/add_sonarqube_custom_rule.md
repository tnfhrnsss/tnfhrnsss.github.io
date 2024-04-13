---
layout: post
title: Add SonarQube Custom Rule
description: Add SonarQube Custom Rule
date: 2024-04-13 20:07:00
last_modified_at : 2024-04-13 20:07:00
parent: SonarQube
grand_parent: Quality Practice
nav_exclude: true
---

# 요구사항
- 산출물 중 테이블 명세서를 자동으로 생성하게 되는데, 그 때 사용할 컬럼 설명으로 @Comment를 활용하고 있다.
- 누락되는 경우를 방지하기 위해 코드 컨벤션 항목으로 한다.

# 환경

- sonarqube 9.9.1
- java 17

# 룰 코드 작성

- 룰
    - Rule 1 : @Table 어노테이션이 선언된 클래스는 반드시 @Comment어노테이션을 통해 테이블 설명이 정의되어야한다.
    - Rule 2: Jpa에 사용될 데이터 모델 Jpo클래스의 멤버 변수는 반드시 @Comment 어노테이션을 통해 컬럼명을 정의해야 한다.

- BaseTreeVisitor를 상속받아서 ClassTree 또는 MethodTree 또는 VariableTree등 다양한 방식으로 코드에 접근한다.
- 멤버 변수에 어노테이션이 있는지 체크하고자 처음에 VariableTree로 체크했더니 로컬은 잘되는데 서버에서는 멤버 변수가 다 코드스멜 대상으로 검출되었다. 그래서 다시 ClassTree로 바꾸고 annotationType으로 필터

```java
@Override
public void visitClass(ClassTree tree) {
    String className = tree.simpleName().name();
    if (className.contains("Jpo")) {
        tree.members().stream()
            .filter(member -> member.is(Tree.Kind.VARIABLE))
            .forEach(member -> {
                if (!hasComment((Tree) member)) {
                    context.reportIssue(this, member, "Member fields of Jpo class must be annotated with @Comment.");
                }
            });
    }
}

private boolean hasComment(Tree tree) {
    List<AnnotationTree> annotations = ((VariableTree) tree).modifiers().annotations();
    for (AnnotationTree annotation : annotations) {
        IdentifierTree idf = (IdentifierTree) annotation.annotationType();
        if (idf.name().equals("EmbeddedId")) {
            return true;
        }
        if (idf.name().equals("Comment")) {
            return true;
        }
    }
    return false;
}
```

- 룰을 설명하는 html과 json파일을 작성한다.
    
    ```java
    <p>@Table을 사용할 경우 @Comment도 작성했는지 검출한다.</p>
    <h2>Noncompliant Code Example</h2>
    <pre>
    @Table // Noncompliant
    public class TicketJpo {
    }
    </pre>
    <h2>Compliant Solution</h2>
    <pre>
    @Table
    @Comment(value = "티켓")
    public class TicketJpo {
    }
    </pre>
    ```
    

# 배포

- 작성한 코드를 jar로 생성하고 `extensions/plugins`  하위에 jar파일 업로드한다.
- 서비스 재시작 ./sonar.sh stop, start
- jar가 적용되었는지 확인하는방법
    - 9000번 콘솔 접속 후 어드민으로 로그인
    - Administration → Marketplace → installed 메뉴 이동하면
    
    ![add_sonarqube_custom_rule1.png](../img/add_sonarqube_custom_rule1.png)
    

- 하단에 **`[Java Custom Rules]`** 에서 pom.xml 버전으로 확인이 가능하다.
- 룰도 추가되었는지 확인한다.
    - 상단 Rules 메뉴로 이동해서 검색

# 적용

- 상단 Rules 메뉴로 이동해서 검색
- 상세화면으로 들어가서 `**Quality Profiles**`에서 activate할 profile을 선택한다.
    - 운영에 바로 적용하면, 오탐 가능성이 있어서 테스트용 Profile을 생성하고 테스트용 프로젝트만 연결했다.
    - 여러팀에서 사용하는 프로필이기 때문에 테스트용 프로필을 복사생성해서 거기에 선 적용하는 방식으로 검증 작업을 진행하고 실적용을 하도록 하자.
    
    ![add_sonarqube_custom_rule2.png](../img/add_sonarqube_custom_rule2.png)
    

# 테스트

- @Comment가 없는 Jpo클래스를 푸시 후 젠킨스를 돌려본다. 🥁🥁
- 테스트 코드를 통해 로컬에서 테스트했지만 서버에 적용하면 오탐이 되는 등 차이가 발생했는데, intellij에 소나큐브 설정을 한 후 해볼 수 있을 것 같다.

# 결과

![add_sonarqube_custom_rule3.png](../img/add_sonarqube_custom_rule3.png)

# Reference

- [adding-coding-rules-using-java](https://docs.sonarsource.com/sonarqube/latest/extension-guide/adding-coding-rules/#adding-coding-rules-using-java)
- [https://ghostitm.tistory.com/69](https://ghostitm.tistory.com/69)