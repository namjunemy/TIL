# Express basic

* expressjs는 nodejs로 만들어진 웹 프레임 워크이다.
* [http://expressjs.com/ko/](http://expressjs.com/ko/) 에 들어가면 이해에 도움이 되는 자세한 자료들이 있다.
* express는 5가지 개념이 있다.
  * 어플리케이션
  * 미들웨어
    * 미들웨어는 함수들의 배열인데, express에 어떤 기능을 추가할 때 마다 미들웨어를 통해서 서버에 기능을 추가 할 것이다.
  * 라우팅
    * 라우팅을 체계적으로 관리할 수 있는 기능을 제공한다.
  * 요청객체
  * 응답객체
    * 요청객체와 응답객체를 한번더 감싸서 편리하게 사용할 수 있도록 메소드 형태로 제공한다.

  

## 어플리케이션

* Express 인스턴스를 어플리케이션이라고 한다.

  ```javascript
  const express = require('express');
  const app = express();
  ```

* 서버에 필요한 기능인 미들웨어를 어플리케이션에 추가한다.

* 라우팅 설정을 할 수 있다.

* 서버를 요청 대기 상태로 만들 수 있다.

  ```javascript
  const express = require('express');
  const app = express();

  app.listen(3000, function() {
    console.log('server is running');
  });
  ```

  

## 미들웨어

* 미들웨어는 함수들의 연속이다.

* 로깅 미들웨어를 만들어 보자

  * 미들웨어를 추가할 때는 use() 라는 함수를 쓴다.
  * 미들웨어는 인터페이스가 정해져 있다. (req, res, next)
  * 미들웨어는 **next() 함수를 호출해야만 다음 로직을 수행**한다.(logger 함수에서 next()를 주석처리 하면 curl 명령어도 결과값을 받아오지 못하고, 콘솔에도 i am logger 만 찍힌다.)
  * `$ npm init`
  * `$ npm install express -s`

  ```javascript
  const express = require('express');
  const app = express();

  function logger(req, res, next) {
    console.log('i am logger');
    next();
  }

  function logger2(req, res, next) {
    console.log('i am logger2');
    next();
  }

  app.use(logger);
  app.use(logger2);

  app.listen(3000, function() {
    console.log('server is running');
  });
  ```

  ```javascript
  server is running
  i am logger
  i am logger2
  ```

* 써드파티 미들웨어를 사용해보자.

  * 이 또한 노드 모듈 형태로 제공한다.
  * 마찬가지로 app.use()를 통해 등록할 수 있다.
  * morgan 미들웨어 사용
  * 파라미터인 dev는 로그의 정도로 설정할 수 있다.

  ```javascript
  const express = require('express');
  const app = express();
  const logger3 = require('morgan');

  function logger(req, res, next) {
    console.log('i am logger');
    next();
  }

  function logger2(req, res, next) {
    console.log('i am logger2');
    next();
  }

  app.use(logger);
  app.use(logger2);
  app.use(logger3('dev'));

  app.listen(3000, function () {
    console.log('server is running');
  });
  ```

  ```javascript
  server is running
  i am logger
  i am logger2
  GET / 404 5.525 ms - 139
  ```

* 일반 미들웨어 vs 에러 미들웨어

  * 위에서 만들었던 미들웨어는 일반 미들웨어이다. 일반 미들웨어는 res, req, next 3가지 인자를 가진다.
  * 하지만 next에 에러가 발생해서 에러객체가 넘어가면, 다음 미들웨어는 인자가 4개 필요하다.
  * 에러 미들웨어의 역할은 에러를 처리하거나, 처리 못했으면 다시 다음 미들웨어 한테 넘기는 역할을 한다.

  ```javascript
  const express = require('express');
  const app = express();

  function commonmw(req, res, next) {
    console.log('i am logger');
    next(new Error('error ouccered'));
  }

  function errormw(err, req, res, next) {
    console.log(err.message);
    next();
  }

  app.use(commonmw);
  app.use(errormw);

  app.listen(3000, function () {
    console.log('server is running');
  });
  ```

  ```javascript
  i am logger
  error ouccered
  ```


* 404, 500 에러 미들웨어

  

## 라우팅

* 요청 url에 대해 적절한 핸들러 함수로 연결해 주는 기능을 라우팅이라고 부른다.
* hello node에서 http기본 모듈을 가지고 라우팅 했던 것을 Express의 어플리케이션에서는 get(), post() 메소드로 깔끔하게 구현할 수 있다.
* 라우팅을 위한 전용 Router 클래스를 사용할 수도 있다. 라우팅 클래스를 이용하면 조금더 복잡한 라우팅을 구현할 수 있다.

  

## 요청 객체

* 클라이언트 요청 정보를 담은 객체를 요청(Request)객체라고 한다.
* http 모듈의 request 객체를 래핑한 것이다.
* req.params(), req.query(), req.body() 메소드를 주로 사용한다.

  

## 응답 객체

* 클라이언트 응답 정보를 담은 객체응 응답(response)객체 라고 한다.
* http 모듈의 response 객체를 래핑한 것이다.
* res.send() - 클라이언트에 문자열 응답, res.status() - http 스테이터스 코드 보내는 것 , res.json() - json 데이터 응답 메소드를 주로 사용한다.



## Hello Express

```javascript
const express = require('express');
const app = express();

//라우팅 경로를 설정한 코드, 여기서 req, res는 http 모듈의 req, res를 래핑한 것이다.
app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});
```

