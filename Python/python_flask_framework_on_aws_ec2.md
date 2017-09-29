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
>
> 학습 참조 링크 :  [https://code.tutsplus.com/ko/tutorials/an-introduction-to-pythons-flask-framework--net-28822](https://code.tutsplus.com/ko/tutorials/an-introduction-to-pythons-flask-framework--net-28822)

 

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

#### testapp/templates/layout.html

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

단순히 일반적인 HTML 파일처럼 보이지만 {% block content %}, {% endblock %}은 생소하다. **home.html**을 통해 이것들의 기능에 대해 알아본다.

#### testapp/templates/home.html

```html
{% extends "layout.html"%}
{% block content %}
  <div class="jumbo">
    <h2>Welcome to the Flask app</h2>
    <h3>This is the home page for the Flask app</h3>
  </div>
{% endblock %}
```

**layout.html**에는 ```content``` 라는 자식 템플릿이 들어가는 블럭이 정의되어 있다. **home.html** 파일은 **layout.html** 의 자식 템플릿으로 여러가지 특징을 상속받습니다. 즉, **layout.html** 은 사이트의 공통 요소를 정의하고 각각의 자식 템플릿은 내용에 맞게 부모 템플릿으로 받은 요소들을 수정해서 사용해야 한다.

이제 브라우저에서 home.html을 볼 수 있도록 URL을 맵핑한다. routes.py를 생성하고 아래와 같이 입력한다.

#### testapp/routes.py

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
  return render_template('home.html')

if __name__ == '__main__':
  app.run(host='0.0.0.0', port=5000, debug=True)
```

위의 코드에 대한 설명은 아래와 같다.

1. 플라스크 클래스와 함수 render_template를 import한다.
2. 플라스크 클래스에 새로운 인스턴스를 만든다.
3. URL ```/```에 함수 home()을 매핑한다. 그러면 URL로 들어왔을때 home()이 실행된다.
4. 함수 home()은 플라스크 함수 render_template()를 사용 하여 template/ 디렉토리에서 home.html을 렌더링하여 브라우저로 보낸다.
5. 마지막으로 run()을 통해 서버에 앱을 실행한다. debug 모드를 true로 설정했기 때문에 오류 메세지를 볼 수 있다. 그리고 코드가 변경되면 서버는 자동으로 다시 로드한다.
6. 추가적으로 app.run()의 host 옵션을 주지 않으면 default는 local서버에 앱을 실행한다. EC2같은 경우 외부접속이 필수 이므로 host옵션을 설정해야 한다.

아래의 명령을 통해서 그동한 작업한 결과를 웹 브라우저에서 확인할 수 있다.

* ```$ python3 routes.py```
* 웹 브라우저에서 http://{EC2 Public DNS}:5000으로 접속한다.

### CSS 적용 

#### testapp/static/css/main.css 생성

```css
body {
  margin: 0;
  padding: 0;
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  color: #444;
}
 
/*
 * Create dark grey header with a white logo
 */
  
header {
  background-color: #2B2B2B;
  height: 35px;
  width: 100%;
  opacity: .9;
  margin-bottom: 10px;
}
 
header h1.logo {
  margin: 0;
  font-size: 1.7em;
  color: #fff;
  text-transform: uppercase;
  float: left;
}
 
header h1.logo:hover {
  color: #fff;
  text-decoration: none;
}
 
/*
 * Center the body content
 */
  
.container {
  width: 940px;
  margin: 0 auto;
}
 
div.jumbo {
  padding: 10px 0 30px 0;
  background-color: #eeeeee;
  -webkit-border-radius: 6px;
     -moz-border-radius: 6px;
          border-radius: 6px;
}
 
h2 {
  font-size: 3em;
  margin-top: 40px;
  text-align: center;
  letter-spacing: -2px;
}
 
h3 {
  font-size: 1.7em;
  font-weight: 100;
  margin-top: 30px;
  text-align: center;
  letter-spacing: -1px;
  color: #999;
}
```

이 스타일 시트를 **layout.html**에 추가해서 자식 템플릿에도 쓸 수 있도록 한다.

```css
<link rel="stylesheet" href"{{ url_for('static', filename='css/main.css') }}">
```

플라스크의 함수 **url_for()**를 사용해서 static디렉토리의 main.css파일의 URL주소를 생성한다. 이제 layout.html은 아래와 같이 변경된다.

```css
<!DOCTYPE html>
<html>
  <head>
    <title>Flask App</title>
    <strong><link rel="stylesheet" href="{{ url_for('static', filename='css/main.css') }}"></strong>
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

다시 웹 사이트에 접근해 보면 CSS가 적용된 웹 페이지를 볼 수 있다. 새로운 페이지를 추가해보자.



## About 페이지 만들기

위에서 웹 템플릿 **home.html**을 만들 때, **layout.html**을 사용해서 만들었다. 그런 다음 URL ```/```가 **home.html**로 연결되도록 **routes.py**를 수정했다. 그리고 css파일을 추가해서 스타일을 입혔다. 다시 반복해서 새로운 about 페이지를 만들어 본다.

#### testapp/templates/about.html

```html
{% extends "layout.html" %}

{% block content %}
  <h2>About</h2>
  <p>This is an About page for the Intro to Flask article. Don't I look good? Oh stop, you're making me blush.</p>
{% endblock %}
```

새로운 페이지에 접속하기 위해서 URL을 새롭게 매핑해야 한다. routes.py 파일을 열고 다음과 같이 매핑을 추가한다.

#### testapp/routes.py

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
  return render_template('home.html')

@app.route('/about')
def about():
  return render_template('about.html')

if __name__ == '__main__':
  app.run(host='0.0.0.0', port=5000, debug=True)
```

URL ```/about```을 함수 **about()**에 매핑한다. 이제 브라우저를 열고 /about으로 접속하면 새로 만든 페이지로 접근할 수 있다. 



## 네비게이션

대부분 웹사이트들은 헤더나 푸터에 메인페이지로 돌아가는 링크가 있다. 이런 링크는 모든 웹사이트의 모든 페이지에서 보여진다. **layout.html**을 열고, 아래의 코드를 추가해서 상속받는 템플릿에서 보여지게 해보자. 정확하게는 \<nav\> 엘리먼트를 \<header\> 코드 안에 넣는다.

#### testapp/templates/layout.html

플라스크 함수인 ```url_for```을 사용해서 URL을 만든다.

```html
...
<header>
  <div class="container">
    <h1 class="logo">Flask App</h1>
    <strong>
      <nav>
        <ul class="menu">
          <li><a href="{{ url_for('home') }}"></a>Home</li>
          <li><a href="{{ url_for('about') }}"></a>About</li>
        </ul>
      </nav>
    </strong>
  </div>
</header>
...
```



####testapp/static/css/main.css

```html
...
.menu {
  float: right;
  margin-top: 8px;
}

.menu li {
  display: inline;
}

.menu li + li {
  margin-left: 35px;
}

.menu li a {
  color: #999;
  text-decoration: none;
}

```



