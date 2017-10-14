# TDD로 하는 API 서버 개발

테스트 주도 개발 학습을 통해서 mocha, should, supertest까지 테스트를 해 보았다. 본격적으로 GET /users API의 스펙을 만들고, 그것에 대한 테스트 코드를 작성한다. 그리고 테스트 코드에 맞게 API의 코드를 개선해 본다.

## NPM 테스트 스크립트

- 기존에 package.json에 추가되어 있던 start 스트립트 외에
- test 스크립트를 추가한다. 이 script에서는 전체경로가 아닌 mocha라고만 설정해줘도 알아서 경로를 찾아서 실행한다.

```javascript
{
  "name": "tdd_api_server",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "mocha app.spec.js",
    "start": "node app.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.16.2",
    "morgan": "^1.9.0"
  },
  "devDependencies": {
    "mocha": "^4.0.1",
    "should": "^13.1.2",
    "supertest": "^3.0.0"
  }
}
```

  

## 사용자 목록 조회 API 테스트 만들기

* 성공
  * 유저 객체를 담은 배열로 응답한다.
  * 최대 limit 갯수만큼 응답한다.
* 실패
  * limit이 숫자형이 아니면 400을 응답한다.
    * limit은 응답받을 데이터의 총 길이
  * offset이 숫자형이 아니면 400을 응답한다.
    * offset은 넘어오는 데이터가 많을 경우에, 잘라서 데이터를 받게 되는데 앞에 몇개 데이터는 skip하고 데이터를 달라고 하는 경우가 있다. 이 때, skip하는 데이터가 offset에 들어간다.

  

### 성공시

* app.spec.js에 GET /users 테스트 코드 작성
  * `$ node_modules/.bin/mocha app.spec.js`		
  * 테스트 성공 결과 확인

```javascript
const app = require('./app');
const request = require('supertest');
const should = require('should');

describe('GET /users는', (done) => {
  describe('성공시', () => {
    it('유저 객체를 담은 배열로 응답한다.', () => {
      request(app)
          .get('/users')
          .end((err, res) => {
            res.body.should.be.instanceOf(Array);
            done();
          });
    });
  });
});
```

```shell
> tdd_api_server@1.0.0 test /Users/njkim/Workspace/intellij/nodejs_api_server_tdd/tdd_api_server
> mocha app.spec.js



Example app listening on port 3000!
  GET /users는
    성공시
      ✓ 유저 객체를 담은 배열로 응답한다.
GET /users 200 3.581 ms - 71


  1 passing (36ms)
```

  

* limit 조건을 테스트 하기 위하여 다음과같이 테스트 코드를 수정한다.

  * 여기서는 limit값을 쿼리스트링 형태로 ?뒤에 넘겨준다.
  * 그리고 should.have.lengthOf() 함수로 결과를 검증한다.
  * app.spec.js코드 수정

  ```javascript
  const app = require('./app');
  const request = require('supertest');
  const should = require('should');

  describe('GET /users는', (done) => {
    describe('성공시', () => {
      it('유저 객체를 담은 배열로 응답한다.', () => {
        request(app)
            .get('/users')
            .end((err, res) => {
              res.body.should.be.instanceOf(Array);
              done();
            });
      });

      it('최대 limit 갯수만큼 응답한다.', () => {
        request(app)
            .get('/users?limit=2')
            .end((err, res) => {
              res.body.should.have.lengthOf(2);
              done();
            });
      })
    });
  });
  ```

  * app.js 코드 수정
  * 요청하는 limit에 따라서 유저배열을 처리하여 넘긴다.

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
    const limit = req.query.limit;
    res.json(users.slice(0, limit));
  });

  app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
  });

  module.exports = app;
  ```

  * 테스트 수행 

  ```shell
  > tdd_api_server@1.0.0 test /Users/njkim/Workspace/intellij/nodejs_api_server_tdd/tdd_api_server
  > mocha app.spec.js



  Example app listening on port 3000!
    GET /users는
      성공시
        ✓ 유저 객체를 담은 배열로 응답한다.
  GET /users 200 3.450 ms - 71
        ✓ 최대 limit 갯수만큼 응답한다.
  GET /users?limit=2 200 0.302 ms - 71


    2 passing (52ms)
  ```

  

### 실패시

* limit의 파라미터로 숫자형이 아닐 경우를 테스트 하기 위한 코드를 작성한다

  * app.spec.js 작성
  * Supertest의 expect() 함수를 통하여 검증할 수 있다.

  ```javascript
  const request = require('supertest');
  const should = require('should');
  const app = require('./app');

  describe('GET /users는', (done) => {
    describe('성공시', () => {
      it('유저 객체를 담은 배열로 응답한다.', () => {
        request(app)
            .get('/users')
            .end((err, res) => {
              res.body.should.be.instanceOf(Array);
              done();
            });
      });

      it('최대 limit 갯수만큼 응답한다.', () => {
        request(app)
            .get('/users?limit=2')
            .end((err, res) => {
              res.body.should.have.lengthOf(2);
              done();
            });
      });
    });

    describe('실패시', () => {
      it('limit이 숫자형이 아니면 400을 응답한다', (done) => {
        request(app)
            .get('/users?limit=two')
            .expect(400)
            .end(done);
      });
    });
  });
  ```

  * app.js 코드 변경
    * limit 변수의 기본값을 주기 위해서 쿼리 스트링으로 limit값이 넘어올 경우 그 값을 쓰고, 기본으로 10을 준다.
    * parseInt를 통해서 숫자를 판단한 결과 숫자형이 아니면 NaN이 넘어오기 때문에 400을 리턴하고 끝낸다.

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
    req.query.limit = req.query.limit || 10;
    const limit = parseInt(req.query.limit, 10);
    if (Number.isNaN(limit)) {
      return res.status(400).end();
    }
    res.json(users.slice(0, limit));
  });

  app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
  });

  module.exports = app;
  ```

  ```javascript
  > tdd_api_server@1.0.0 test /Users/njkim/Workspace/intellij/nodejs_api_server_tdd/tdd_api_server
  > mocha app.spec.js



  Example app listening on port 3000!
    GET /users는
      성공시
        ✓ 유저 객체를 담은 배열로 응답한다.
  GET /users 200 3.367 ms - 71
        ✓ 최대 limit 갯수만큼 응답한다.
  GET /users?limit=2 200 0.566 ms - 47
      실패시
  GET /users?limit=two 400 0.280 ms - -
        ✓ limit이 숫자형이 아니면 400을 응답한다


    3 passing (50ms)

  ```

  ​

  ​