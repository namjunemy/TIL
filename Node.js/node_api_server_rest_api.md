# REST API

## HTTP 요청

* HTTP 메소드
  * 서버 자원에 대한 행동을 나타낸다.(동사로 표현)
    * GET : 자원을 조회
    * POST : 자원을 생성
    * PUT : 자원을 갱신
    * DELETE : 자원을 삭제
  * 이는 Express 어플리케이션의 메소드로 구현되어 있고, 이미 우리는 app.get()을 통해서 사용했었다.


* user1에 대한 요청의 예시

  * `$ curl -X GET 'localhost:3000/users/1'`

* GET 요청 예시

  ```javascript
  app.get('/users', function (req, res) {
    res.send('users list');
  });
  ```

* POST 요청 예시

  ```javascript
  app.post('/users', function (req, res) {
    //유저 생성 로직
    res.send(user);
  });
  ```

  

## HTTP 응답

* HTTP 상태코드
* 가장 많이 쓰이는 상태코드는 200번 대와 400번 대 이다.
  * 1XX: 아직 처리중
  * 2XX: 자, 여기 있어!
    * 200 : 성공(success), GET, PUT
    * 201 : 작성됨(created), POST
    * 204 : 내용 없음(No Content), DELETE
  * 3XX: 잘 가~
  * 4XX: 클라이언트 쪽 문제
    * 400 : 잘못된 요청(Bad Request)
    * 401 : 권한 없음(Unauthorized)
    * 404 : 찾을 수 없음(Not found)
    * 409 : 충돌(Conflict). 이미 있는 자원 생성시.
  * 5XX: 서버 쪽 문제
    * 500 : 서버 에러(Internel server error)

  

## 첫번째 API 만들기: 사용자 목록 조회 API

* GET /users

* 사용자 목록을 조회하는 기능

  ```javascript
  const express = require('express');
  const app = express();
  const morgan = require('morgan');

  let users = [
    {id: 1, name: 'alice'},
    {id: 2, name: 'bek'},
    {id: 3, name: 'chris'}
  ];

  app.use(morgan('dev'));

  app.get('/users', function (req, res) {
    res.json(users);
  });

  app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
  });
  ```

  * curl 또는 브라우저 직접 접속해서 확인

    * 자세히 보기 위해 v옵션 사용
    * curl -X GET 'localhost:3000/users' -v
    * 결과 확인(상태코드 200)
      * \> : 오른쪽 화살표는 요청에 대한 정보
      * \< : 왼쪽 화살표는 응답에 대한 내용
    * res.json() 메소드를 사용했기 때문에, Content-Type이 application/json으로 되어 있는 것을 볼 수 있다.

    ```javascript
    $ curl -X GET 'localhost:3000/users' -v
    Note: Unnecessary use of -X or --request, GET is already inferred.
    *   Trying ::1...
    * TCP_NODELAY set
    * Connected to localhost (::1) port 3000 (#0)
    > GET /users HTTP/1.1
    > Host: localhost:3000
    > User-Agent: curl/7.54.0
    > Accept: */*
    > 
    < HTTP/1.1 200 OK
    < X-Powered-By: Express
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 71
    < ETag: W/"47-AswcodMnmv7+S9S8PW5fCVVRgCc"
    < Date: Wed, 11 Oct 2017 16:17:13 GMT
    < Connection: keep-alive
    < 
    * Connection #0 to host localhost left intact
    [{"id":1,"name":"alice"},{"id":2,"name":"bek"},{"id":3,"name":"chris"}]
    ```

    ​