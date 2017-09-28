# Python Flask Framework on AWS EC2

> 설치환경
>
> OS : Ubuntu 16.04 LTS(AWS EC2 리눅스 인스턴스)
>
> [EC2 인스턴스 생성](https://github.com/namjunemy/TIL/blob/master/AWS/aws_ec2_create_instance.md)
>
> [EC2 원격 접속](https://github.com/namjunemy/TIL/blob/master/AWS/aws_ec2_connect_and_scp.md)
>
> [EC2에 Jupyter Notebook 개발환경 설치](https://github.com/namjunemy/TIL/blob/master/Python/python_jupyter_notebook_on_aws_ec2.md) 

 

플라스크 작고 강력한 웹 프레임 워크이다. 짧은 시간에 웹앱을 만들 수 있다. 정적인 페이지를 서비스하는 간단한 웹사이트를 만들어 본다. 플라스크 웹 프레임 워크가 어떻게 동작하는지 알기위한 목적이다.  

## 진행전 주의 사항

* 진행 하기 이전에 Jupyter Notebook에서 생성되는 .ipynb파일을 .py파일로 변경할 필요가 있을 때가 있다. 아래의 명령을 통해서 .py파일로 변환할 수 있다.
  * ```$ jupyter nbconvert --to script [filename].ipynb  ```
* AWS EC2의 Security Group 설정 중 inbound 규칙에 Flask의 기본 포트인 5000을 추가한다.
  * Type : Custom TCP / Protocol : TCP / Port Range : 5000 / Source : 0.0.0.0/0 

   

## 플라스크 설치

먼저 플라스크를 설치한다. 위에서 Jupyter Notebook 개발환경 설치 링크의 가이드를 통해 개발환경을 설치했다면 아래의 명령을 통해 설치 할 수 있다.

* ```$ sudo pip3 install Flask```

  

## 프로젝트 구조 만들기

아래와 같이 flaskapp/ 안에 프로젝트 구조를 생성한다.

* flask/
  * testapp/
    * static/
      * css/
      * img/
      * js/
    * templates/
    * routes.py
    * README.md 

flask/ 내에 모든 파일을 포함하고 있는 testapp/ 디렉토리를 만든다. testapp/ 디렉토리 안에 static/ 디렉토리를 만든다. 이곳에는 앱에 필요한 이미지와 CSS파일 그리고 JS파일이 들어가게 된다. 추가적으로 templates/ 디렉토리를 만들어 앱에서 사용될 탬플릿을 저장한다. route.py를 만들고 URL 라우팅 코드를 작성할 것이다. 



### 작업 흐름

1. 도메인의 root URL ```/``` 로 접속하면 홈페이지로 이동할 것이다.
2. routes.py는 URL ```/``` 에서 파이썬 플라스크를 실행한다.
3. 플라스크는 templates/ 디렉토리 안의 템플릿을 찾는다.
4. 템플릿은 static/ 디렉토리 안에서 HTML페이지를 표시하기 위한 이미지 파일, CSS, 자바스크립트 파일을 찾는다.
5. 생성된 HTML 페이지는 routes.py 로 리턴된다.
6. routes.py가 브라우저에 HTML을 보낸다.



## 홈페이지 생성(feat. 웹 템플릿)

반복되는 페이지를 생성하는 것은 지루하다. 게다가 새로운 CSS 파일을 만들어야 한다면 더욱 더 귀찮다. 모든 CSS 코드를 새로 추가해야 되기 때문이다. 반복적인 HTML문을 작성하는 것 보다 하나의 페이지 레이아웃을 정해놓고 필요할때마다 내용을 집어 넣는 다면 아주 편리하다. 그것이 바로 웹 템플릿이 하는 일이다.

웹 템플릿이 사용될 때 마다 변수들은 내용에 따라서 변경된다. 웹 템플릿은 반복적인 일을 없애고, 내용을 디자인으로 부터 분리해서 앱을 관리하기 쉽게 만들어 준다. 플라스크는 Jinja2 템플릿 엔진을 사용한다.

첫번쨰로 HTML 문서의 뼈대로 사용될 layout.html 파일을 만들어서 templates/ 디렉토리에 넣는다.

#### app/templates/layout.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Flask App</title>
  </head>
  <body>
    <header>
      <div class="container">
        <h1 class="logo">Flask App</h1>
      </div>
    </header>
    
    <div class="container">
      {% block content %}
      {% endblock %}
    </div>
  </body>
</html>
```

단순히 일반적인 HTML 파일처럼 보이지만 {% block content %}, {% endblock %}은 생소하다. home.html을 통해 이것들의 기능에 대해 알아본다.

#### app/templates/home.html

```html
{% extends "layout.html"}
{% block content %}
  <div class="jumbo">
    <h2>Welcome to the Flask app</h2>
    <h3>This is the home page for the Flask app</h3>
  </div>
{% endblock %}
```

