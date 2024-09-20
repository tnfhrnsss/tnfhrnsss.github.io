---
layout: post
title: Python과 Flask로 AI 애플리케이션 개발하기
date: 2024-09-18 08:31:00
last_modified_at : 2024-09-18 08:31:00
parent: Coursera
grand_parent: Mooc
nav_exclude: true
---

# 강의 정보

- [코세라 강의 주소](https://coursera.org/share/bda1ab1ed10e7be7ca8e53a6c8794d0e){:target="_blank"}  
- 수강료: 7일내에 수강 완료한다면 **무료**
- 수강 시간: 약 11시간
- 과정
    - 애플리케이션의 함수 생성
    - 애플리케이션 함수 패키지화
    - 함수를 패키지로 호출하여 함수에 대한 단위 테스트 실행
    - Flask를 사용하여 웹을 통해 애플리케이션 배포하기
    - 애플리케이션에 오류 처리 기능 통합
    - 생성된 코드에 대해 정적 코드 분석 실행

# 환경

- python 3.7+
- pylint (과제 수행시 필요)
- flask 2.2.2
- 개발 IDE
    - cognitiveclass에서 제공하는 cloud ide로 수강하고 과제 수행
    - watson ai 라이브러리가 로드되어 있어서, 별도로 환경 구성하지 않아도 됩니다.
    

# 수강 후기

- 로컬 환경이 별로 좋지 않아서 cloud ide는 좋았습니다.
- 마지막 과제 진행 중에 파일이 날아가서 다시 첨부터 코딩해야 했습니다.ㅠㅜ 두번째부터는 깃헙에 푸시하면서 했습니다.
- 항상 python 기본기가 부족하다는 생각이 들었었는데, 이 강좌로 기본에 대해 학습할 수 있었습니다.
- 무엇보다 프로젝트 구조가 어떻게 구성되는지 알수 있어서 개인적으로 유익했습니다.
- 수준은 중급이라고 표시되어있지만, 파이썬 기본을 안다면 초급이여도 가능하다고 봅니다.

# 과제 or 테스트

- 과정 중간에 보는 객관식 테스트는 영어로 볼 것을 추천합니다. (한글은 번역되다보니 잘 이해가 안되는 부분이 있습니다.)
- 마지막 과제는 IBM Watson 라이브러리를 사용하여 AI 기능이 통합된 웹 앱을 개발하는 건데요. 수업만 잘 따라왔다면 전혀 어렵지 않습니다.
- AI가 과제채점을 하고 며칠 후 결과가 메일로 통보됩니다.
- 마지막으로 동료 채점을 2개 완료해야 과정을 완료할 수 있습니다.

# python 기본

### 1. python 코딩 스타일

- [PEP8](https://peps.python.org/pep-0008/){:target="_blank"}  

### 2. unit test

```bash
import unittest

from mymodule import square, double

class TestModule(unittest.TestCase): 

    def test1(self): 
        self.assertEqual(square(2), 4)
```

### 3. module

- 파이썬 정의, function, class를 포함하는 py파일이다.
- import해서 다른 스크립트에서 호출할 수 있다.
    
    ```bash
    from module import square, doubler
    
    def test():
    	sqaure(4)
    	doubler(4)
    ```
    

### 4. package

- python 모듈을 **init**.py파일이 있는 딕셔너리로 모은 collection 단위이다
    - 패키지로 만들러면 해당 폴더에 __init__.py파일이 있어야한다.
    - __init__.py 파일에 패키지에 필요한 모듈을 참조하는 코드를 추가한다.
        
        ```bash
        from . import module1
        from . import module2
        ```
        
- 모듈이나 패키지를 가져올때 파이썬으로 만든 해당 객체는 항상 module유형이다
- 모듈과 패키지는 파일 시스템 수준에서만 구분된다.
- 패키지가 같은 디렉터리에 있으면 다른 스크립트에서도 사용할 수 있다.
    
    ```bash
    from myproject.module1 import square, doubler
    from myproject.module2 import mean
    ```
    
- verify 과정을 거친다
    - python 모듈 실행하고
    - import 패키지 명을 실행했을 때 오류없이 실행되면 정상적으로 로드되었음을 의미한다.

### 5. library

- 라이브러리는 패키지 컬렉션이거나 단일 패키지일 수 있다.
- 패키지와 라이브러리는 혼용되어 같은 의미로 사용될 수 있다.

# Flask

## 1. 설치

- python virtual 생성
    
    ```bash
    python3 -m venv venv
    source ./venv/bin/activate
    ```
    
- install flask
    
    ```bash
    pip install Flask==2.2.2
    ```
    
- 참고) 내장 모듈 확인하는 방법
    
    ```bash
    pip freeze
    ```
    

## 2. Flask vs Django

- flask가 매우 가벼운 프레임워크인반면에 django는 풀스택 프레임워크다.
- django에 비해 flask는 매우 유연한다.

## 3. 정리

### 1. run

```bash
flask --app server --debug run
```

- 옵션
    - —app : 실행할 python 파일을 식별해서 플라스크 명령에 구성 전달
    - —debug : 디버그 모드로 띄우기

### 2. configuration

- 종류
    - env : 앱이 실행되는 환경을 나타내는 것으로 운영환경인지 개발환경인지
    - debug : 디버그 모드 활성화
    - testing : 테스트 모드 활성화
    - secret_key : 세션쿠키에 서명하는데 사용
- 주입 방식
    - json파일에 저장
        
        ```bash
        app.config.from_file("jsonfile path")
        ```
        
    - dictionary객체에 할당
    
    ```bash
    app.config['SECRET_KEY'] = "random-secret-key"
    ```
    
    - env가 이미 있으면 그 개체에 로드시킨다.
        
        ```bash
        app.config["VARIABLE_NAME"]
        app.config.from_prefixed_env()
        ```
        

### 3. structure

```bash
- config.json
- requirements.txt
- setup.py
- src
--__ini__.py
--static
---css
----main.css
---img
----header.png
---js
----site.js
--templates
----about.html
----index.html
-tests
--test_auth.py
--test_site.py
-venv
```

### 4. api 호출

- custom routes
    
    ```bash
    @app.route("/health")
    
    def health():
    	return jsonify(dict(status="OK")), 200
    ```
    
    ```bash
    @app.route("/health", methods=["GET"])
    
    def health():
    	return jsonify(dict(status="OK")), 200
    ```
    
- Flask.Request class
    - Request
    
    ```bash
    from flask import Flask, request
    app = Flask(__name__)
    
    @app.route('/')
    def hello_workd():
    	course = request.args["course"]
    	rating = request.args.get("rating")
    	return {"message": f"{course} with rating {rating}"}
    ```
    

### 5. json.load

- api 결과를 text가 아닌 json형태로 리턴
    
    ```bash
    import requests
    import json
    
    def sentiment_analyzer(text_to_analyse):
        response = requests.post(url, json=myobj, headers=header)
    
        formatted_response = json.loads(response.text)
    ```
    

### 6. dynamic routes

```bash
from flask import Flask, escape
import requests
```

- example
    - @app.route(”terminals/<string:airport_code>”)
    - @app.route(`/book/<int:isbn>’)

### 7. error handle

```bash
@ app.errorhandler(500)
def server_error(error):
    return {"message": "Something went wrong on the server"}, 500
```

### 8. jsonify_decorator

```bash
def jsonify_decorator(function):
    def modifyOutput():
        return {"output":function()}
    return modifyOutput
    
@jsonify_decorator
def hello():
    return 'hello world'
    
@jsonify_decorator
def add():
    num1 = input("Enter a number - ")
    num2 = input("Enter another number - ")
    return int(num1)+int(num2)
    
print(hello())
print(add())
```

### 9. accessing form data with flask.request.form

```bash
<form method="POST" action="/login">
    <input type="text" name="username">
    <input type="password" name="password">
    <input type="submit" value="Submit">
</form>

///////////////////////////////

from flask import request

@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    # process login here
```

### 10. flask.redirect

```bash
from flask import redirect

@app.route('/admin')
def admin():
    return redirect('/login')
```

### **11. Dynamic URLs with flask.url_for**

```bash
from flask import url_for

@app.route('/admin')
def admin():
    return redirect(url_for('login'))
    
@app.route('/login')
def login():
    return "<Login Page>"
```

### **12. Handling different HTTP request types**

```
@app.route('/data', methods=['GET', 'POST'])
    def data():    
        if request.method == 'POST':        
          # process POST request    
        if request.method == 'GET':        
          # process GET request
```

```
@app.route('/read/<int:id>', methods=['GET'])
  def read(id):    
  # Get the record by id    
  record = get_record(id)    
  return render_template('read.html', record=record)
```

### 

### 13. redirect

```
return redirect(url_for('home'))
redirect(url_for('read', id=record.id))   
```

# 과제 snapshot

![python_flask_ai_application.png](../img/python_flask_ai_application.png)