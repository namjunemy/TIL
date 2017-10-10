# Hello World Node.js 버전

  

* [https://nodejs.org/en/docs/](https://nodejs.org/en/docs/) 링크로 이동하면 최신 버전의 API 문서를 만날 수 있다.
* Usage & Example로 이동하면 다음과 같은 Hello world코드가 있다.

```javascript
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

* 실행 명령은 
  * `$ node example.js`
* 잘 실행 되는지 확인해보려면
  * 직접 브라우저로 해당 url에 접근하거나.
  * curl 명령을 통해 접근한다.
    * Curl -X GET 'localhost:3000'

  

## 라우팅 추가하기

* req 객체를 통해서 클라이언트가 요청한 url 정보를 얻을 수 있다.
  * `req.url` 를 통해서 url 정보를 가져와서 분기를 만들어 라우팅을 추가한다.
  * 하지만, 이럴 경우 분기가 매우 많이 생성되므로 기본 모듈인 http모듈 외에 express 모듈을 사용할 것이다.

```javascript
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  if(req.url === '/') {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello World\n');
  } else if(req.url === '/users') {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('User list\n')
  } else {
    req.statusCode = 404;
    res.end('Not found\n');
  }
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

