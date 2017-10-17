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

  

## 1. 사용자 목록 조회 API 테스트 만들기

* 요구사항
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
  ```



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

## 2. 사용자 조회 API - GET /users/:id

유저 한명을 조회하는 API를 작성한다.

* 요구사항
  * 성공시
    * id가 1인 유저 객체를 반환한다.
  * 실패시
    * id가 숫자가 아닐경우 400으로 응답한다.
    * id로 유저를 찾을 수 없는 경우 404로 응답한다.



​    

### 성공시

* app.spec.js 테스트 코드 작성
  * `res.body.should.have.property()` 함수는 첫번째 파라미터로 기대하는 id 문자열을 넣어주고, 두번째 파라미터로는  id의 값이 1이다 라는 것을 검증해주는 값이 들어간다.

```javascript
const app = require('./app');
const request = require('supertest');

describe('GET /users는', (done) => {
  it('슈퍼테스트 실행', () => {
    request(app)
        .get('/users')
        .end((err, res) => {
          console.log(res.body);
          done();
        });
  });
});

describe('GET /users/1', () => {
  describe('성공시', () => {
    it('id가 1인 유저 객체를 반환한다', (done) => {
      request(app)
          .get('/users/1')
          .end((err, res) => {
            res.body.should.have.property('id', 1);
            done();
          });
    });
  });
});
```

* 테스트 케이스에 통과할 수 있도록 app.js 코드도 수정한다.
  * req.params.id를 통해서 요청받는 id정보를 받을 수 있다.
  * users 배열의 filter함수를 통해서 user의 id가 요청받은 id인 객체들을 반환하고, 그 배열의 0번 인덱스에 접근하여 해당 유저를 얻는다.
  * 이 유저를 json으로 반환한다.

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

app.get('/users/:id', function (req, res) {
  const id = parseInt(req.params.id);
  const user = users.filter((user) => user.id === id)[0];
  res.json(user);
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});

module.exports = app;
```

* 테스트 성공 결과 값

```shell
> tdd_api_server@1.0.0 test /Users/njkim/Workspace/intellij/nodejs_api_server_tdd/tdd_api_server
> mocha app.spec.js


Example app listening on port 3000!
  GET /users는
    성공시
      ✓ 유저 객체를 담은 배열로 응답한다.
GET /users 200 2.782 ms - 71
      ✓ 최대 limit 갯수만큼 응답한다.
GET /users?limit=2 200 0.344 ms - 47
    실패시
GET /users?limit=two 400 0.198 ms - -
      ✓ limit이 숫자형이 아니면 400을 응답한다

  GET /users/1
    성공시
GET /users/1 200 1.707 ms - 23
      ✓ id가 1인 유저 객체를 반환한다

  4 passing (61ms)
```

  

### 실패시

* app.spec.js 테스트 코드 수정
  * api호출시 id 파라미터의 값이 숫자가 아닐경우를 테스트 하기 위한 첫번째 테스트 케이스(it() 함수) 생성
  * 요청 파라미터에 해당하는 유저의 조회가 안 될 경우를 테스트하기 위한 두번째 테스트 케이스 생성

```javascript
...(앞 코드 생략)

describe('GET /users/1', () => {
  describe('성공시', () => {
    it('id가 1인 유저 객체를 반환한다', (done) => {
      request(app)
          .get('/users/1')
          .end((err, res) => {
            res.body.should.have.property('id', 1);
            done();
          });
    });
  });

  describe('실패시', () => {
    it('id가 숫자가 아닐경우 400을 응답한다', (done) => {
      request(app)
          .get('/users/one')
          .expect(400)
          .end(done);
    });

    it('id로 유저를 찾을 수 없을 경우 404로 응답한다', (done) => {
      request(app)
          .get('/users/999')
          .expect(404)
          .end(done);
    });
  })
});
```

* 테스트 케이스를 만족하기 위한 app.js 코드 수정
  * 아이디가 숫자형이 아닌 경우 상태코드 400을 리턴라기 위해 id를 Number.isNan()으로 판단한다.
  * user의 조회가 실패할 경우 user는 undefined이므로 이 경우 404를 리턴하도록 한다.

```javascript
...(생략)

app.get('/users/:id', function (req, res) {
  const id = parseInt(req.params.id);
  if (Number.isNaN(id))
    return res.status(400).end();

  const user = users.filter((user) => user.id === id)[0];
  if (!user)
    return res.status(404).end();

  res.json(user);
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});

module.exports = app;
```

* 테스트 케이스 성공 결과

```javascript
> tdd_api_server@1.0.0 test /Users/njkim/Workspace/intellij/nodejs_api_server_tdd/tdd_api_server
> mocha app.spec.js

Example app listening on port 3000!
  GET /users는
    성공시
      ✓ 유저 객체를 담은 배열로 응답한다.
GET /users 200 7.951 ms - 71
      ✓ 최대 limit 갯수만큼 응답한다.
GET /users?limit=2 200 0.608 ms - 47
    실패시
GET /users?limit=two 400 0.134 ms - -
      ✓ limit이 숫자형이 아니면 400을 응답한다

  GET /users/1
    성공시
GET /users/1 200 0.539 ms - 23
      ✓ id가 1인 유저 객체를 반환한다
    실패시
GET /users/one 400 0.068 ms - -
      ✓ id가 숫자가 아닐경우 400을 응답한다
GET /users/999 404 0.062 ms - -
      ✓ id로 유저를 찾을 수 없을 경우 404로 응답한다


  6 passing (78ms)
```

  

## 3. 사용자 삭제 API - DELETE /users/:id

id를 입력하면 해당 id의 유저를 삭제하는 API이다.

* 요구사항
  * 성공시
    * 204를 응답한다.
  * 실패시 
    * id가 숫자가 아닐 경우 400(Bad Request)을 응답한다. 


### 성공시

* 이번에는 삭제로직 이므로, supertest객체(변수 request)의 delete() 함수를 통해서 요청한다.
* expect() 함수를 통해 204 코드를 검증하면 테스트 케이스는 통과된다.
* app.spec.js 코드

```javascript
...(생략)

describe('DELETE /users/1은', () => {
  describe('성공시', () => {
    it('204를 응답한다', (done) => {
      request(app)
          .delete('/users/1')
          .expect(204)
          .end(done);
    });
  });
});
```

* app.js 코드
  * app의 get() 함수가아닌 delete 함수로 삭제요청을 한다.
  * 요청이 들어오면 마찬가지로 req.params.id에 접근하여 id를 가져오고
  * users의 필터함수를 통해 기존에 배열에 있던 user의 id가, 요청한 id 가 아닌 유저들을 배열로 만들어서 users로 대체한다.
  * 그다음 204를 리턴한다.

```javascript
...(생략)

app.delete('/users/:id', function (req, res) {
  const id = parseInt(req.params.id);
  users = users.filter(user => user.id !== id);
  res.status(204).end();
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});

module.exports = app;
```

* 테스트 케이스 성공 결과

```javascript
...(생략)


DELETE /users/1은
    성공시
DELETE /users/1 204 0.159 ms - -
      ✓ 204를 응답한다

  7 passing (62ms)

```

  

### 실패시 

* id가 숫자가 아닐경우를 테스트 하기 위한 코드를 추가한다.
* request의 delete 함수를 통해서 숫자가 아닌 문자 파라미터를 넘긴다.
* app.spec.js 코드

```javascript
...

describe('DELETE /users/1은', () => {
  describe('성공시', () => {
    it('204를 응답한다', (done) => {
      request(app)
          .delete('/users/1')
          .expect(204)
          .end(done);
    });
  });

  describe('실패시', () => {
    it('id가 숫자가 아닐경우 400을 응답한다', (done) => {
      request(app)
          .delete('/users/one')
          .expect(400)
          .end(done);
    });
  });
});
```

* app.spec.js 코드
  * 마찬가지로 id가 NaN이 들어오는 경우를 처리하는 로직을 만든다.
  * 400을 리턴하도록 한다.

```javascript
...

app.delete('/users/:id', function (req, res) {
  const id = parseInt(req.params.id);
  if(Number.isNaN(id))
    return res.status(400).end();
  users = users.filter(user => user.id !== id);
  res.status(204).end();
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});

module.exports = app;
```

* 테스트 케이스 검증 결과

```shell
...

  DELETE /users/1은
    성공시
DELETE /users/1 204 0.290 ms - -
      ✓ 204를 응답한다
    실패시
DELETE /users/one 400 0.049 ms - -
      ✓ id가 숫자가 아닐경우 400을 응답한다

  8 passing (65ms)
```

  

## 4. 사용자 추가 API - POST /users

* 요구사항
  * 성공시
    * 201 상태코드를 반환한다.
    * 생성된 유저 객체를 반환한다
    * 입력한 name을 반환한다
  * 실패시
    * name 파라미터 누락시 400을 반환한다.
    * name이 중복일 경우 409를 반환한다.


### 성공시

* mocha의 before라는 함수로 테스트 케이스가 실행되기 전에 진행되야 할 작업들을 정의한다.
* 여기서는 post요청을 통해 daniel을 생성하고 상태코드로 201을 체크한다. 
* 그리고 유저 객체 반환과 이름을 반환하는지 검증하기 위해 body를 저장하고, name을 미리 정의한다.
* app.spec.js 테스트 코드 작성

```javascript
...

describe('POST /users는', () => {
  describe('성공시', () => {
    let name = 'daniel', body;
    before((done) => {
      request(app)
          .post('/users')
          .send({name: daniel})
          .expect(201)
          .end((err, res) => {
            body = res.body;
            done();
          });
    });
    it('생성된 유저 객체를 반환한다', () => {
      body.should.have.property('id');
    });
    it('입력한 name을 반환한다', () => {
      body.should.have.property('name', name);
    })
  });
});
```

* app.js 코드 작성

  * 새로운 유저를 만들기 위해서 사용자가 입력한 데이터에 접근해야 한다.
  * post요청은 body를 통해서 그 데이터를 넘긴다.
  * 그렇기 때문에 body값에 접근을 해야한다.
  * 따라서 req.body.name과 같이 접근을 해야하는데,
  * express에서는 body를 지원하지 않는다.
  * 따라서, body를 사용하려면 추가적인 모듈을 사용해야한다.
  * body-parser와 multer라는 모듈을 쓴다.
    * `$ npm i body-parser --save`
    * multer는 이미지와 같은 큰 데이터를 사용할때 쓴다.
  * body-parser모듈을 추출해서, app.use() 를 통해 미들웨어 추가를 해준다.
    * application/json과 application/x-www-form-urlencoded 를 파싱하기 위해 미들웨어를 추가한다.

  ```
  ...

  const bodyParser = require('body-parser');

  app.use(bodyParser.json());
  app.use(bodyParser.urlencoded({extended: true}));

  ...
  ```

  * 그 후 다음과 같이 app.js의 코드를 변경할 수 있다.

```javascript
...

app.post('/users', (req, res) => {
  const name = req.body.name;
  const id = users.length + 1;
  const user = {id, name};
  users.push(user);
  res.status(201).json(user);
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});

module.exports = app;
```

* 테스트 코드 검증 결과

```shell
...

  POST /users는
    성공시
POST /users 201 16.374 ms - 24
      ✓ 생성된 유저 객체를 반환한다
      ✓ 입력한 name을 반환한다

  10 passing (103ms)
```

  

### 실패시

* name 파라미터 누락시 400 반환
* name 중복일 경우 409 반환
* 실패시의 app.spec.js 테스트 코드를 추가하면 다음과 같다.
  * 아무것도 없는 빈 객체를 넘겨보고 400 상태코드를 반환하는 것을 검증하고
  * 이미 존재하는 이름으로 생성요청을 하고 409를 반환하는지 검증한다.

```javascript
...

describe('POST /users는', () => {
  describe('성공시', () => {
    let name = 'daniel', body;
    before((done) => {
      request(app)
          .post('/users')
          .send({name: 'daniel'})
          .expect(201)
          .end((err, res) => {
            body = res.body;
            done();
          });
    });
    it('생성된 유저 객체를 반환한다', () => {
      body.should.have.property('id');
    });
    it('입력한 name을 반환한다', () => {
      body.should.have.property('name', name);
    })
  });
  describe('실패시', () => {
    it('name 파라미터 누락시 400을 반환한다', (done) => {
      request(app)
          .post('/users')
          .send({})
          .expect(400)
          .end(done);
    });
    it('name이 중복일 경우 409를 반환한다', (done) => {
      request(app)
          .post('/users')
          .send({name: 'daniel'})
          .expect(409)
          .end(done);
    });
  });
});
```

* 다음은 테스트 코드 검증을 통과하기 위한 app.js 코드이다
  * name 값을 가져와서 이 값이 없다면, 400 코드를 리턴한다. 간단하다.
  * 다음으로 users.filter함수를 이용해서, 요청받은 이름이랑 같은 name인 객체들을 찾고, 그 배열의 length를 저장한다.
  * 그 배열의 length가 0이 아니면 중복객체가 있다는 뜻이므로, 조건문으로 처리하여 409를 리턴한다.

```javascript
...

app.post('/users', (req, res) => {
  const name = req.body.name;
  if (!name)
    return res.status(400).end();

  const isConflict = users.filter(user => user.name === name).length;
  if(isConflict)
    return res.status(409).end();

  const id = Date.now() + 1;
  const user = {id, name};
  users.push(user);
  res.status(201).json(user);
});

...
```

* 테스트 코드 검증 결과

```shell
...

  POST /users는
    성공시
POST /users 201 15.745 ms - 24
      ✓ 생성된 유저 객체를 반환한다
      ✓ 입력한 name을 반환한다
    실패시
POST /users 400 0.589 ms - -
      ✓ name 파라미터 누락시 400을 반환한다
POST /users 409 0.393 ms - -
      ✓ name이 중복일 경우 409를 반환한다

  12 passing (117ms)
```

  

## 5. 사용자 수정 API - PUT /users/:id

* 요구사항
  * 성공시
    * 변경된 name을 응답한다.
  * 실패시
    * 정수가 아닌 id일 경우 400 응답
    * name이 없을 경우 400 응답
    * 없는 유저일 경우 404 응답
    * 이름이 중복일 경우 409응답


### 성공시

* 기존에 있던 4번 유저의 이름을 den으로 변경한다.
* put메소드로 name 값을 설정하여 body에 실어 보낸다.
* 그리고 res의 body값으로 name이 리턴되는지 확인한다.
* app.spec.js 테스트 코드 수정

```javascript
...

describe('PUT /users/:id', () => {
  describe('성공시', () => {
    it('변경된 name을 응답한다', (done) => {
      const name = 'den';
      request(app)
          .put('/users/3')
          .send({name})
          .end((err, res) => {
            res.body.should.have.property('name', name);
            done();
          });
    });
  });
});
```

* app.js 코드 수정

```javascript
...

app.put('/users/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  const name = req.body.name;

  const user = users.filter((user) => user.id === id)[0];
  user.name = name;

  res.json(user);
});

...
```

* 테스트 코드 실행 결과

```javascript
...

app.put('/users/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  const name = req.body.name;

  const user = users.filter((user) => user.id === id)[0];
  user.name = name;

  res.json(user);
});

...
```

  

### 실패시

* 실패할 경우의 테스트 케이스 추가
* 요구사항의 조건대로 각 케이스를 추가한다.
* app.spec.js의 테스트 코드

```javascript
...

describe('실패시', () => {
    it('정수가 아닌 id일 경우 400을 응답한다', (done) => {
      request(app)
          .put('/users/one')
          .expect(400)
          .end(done);
    });
    it('name이 없을 경우 400을 응답한다', (done) => {
      request(app)
          .put('/users/1')
          .expect(400)
          .end(done);
    });
    it('없는 유저일 경우 404를 응답한다', (done) => {
      request(app)
          .put('/users/999')
          .send({name: 'foo'})
          .expect(404)
          .end(done);
    });
    it('이름이 중복일 경우 409를 응답한다', (done) => {
      request(app)
          .put('/users/3')
          .send({name: 'bek'})
          .expect(409)
          .end(done);
    });
  });

..
```

* app.js의 어플리케이션 코드
  * 아이디가 정수가 아닌 경우를 처리하기 위한 첫번째 if문
  * 이름이 없는 경우를 처리하기 위한 두번째 if문
  * users 배열의 필터를 활용하여 수정을 요청한 이름의 데이터가 중복될 경우를 처리하기 위한 세번째 if문
  * 요청한 name에 해당하는 데이터가 없을 경우를 처리하기위한 네번째 if문

```javascript
...

app.put('/users/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  if (Number.isNaN(id))
    return res.status(400).end();

  const name = req.body.name;
  if (!name)
    return res.status(400).end();

  const isConfilct = users.filter(user => user.name === name).length;
  if (isConfilct)
    return res.status(409).end();

  const user = users.filter((user) => user.id === id)[0];
  if (!user)
    return res.status(404).end();

  user.name = name;

  res.json(user);
});

...
```





