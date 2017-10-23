# 데이터베이스

* SQL
  * 테이블 형식의 데이터베이스
  * MySQL, PostgreSQL, Aurora, Sqlite
  * 학습에서는 Sqlite를 사용한다.
* NoSQL
  * MongoDB, DynamoDB
* In Memory DB
  * 말 그대로 메모리 안에 데이터가 저장이 된다. 즉, 휘발성을 가지고 있다. 그런데도 퍼포먼스가 좋기 때문에 사용한다.
  * Redis, Memcashed


  

## SQL 쿼리 기초

* 삽입
  * insert users ('name') values ('alice');
* 조회
  * select * from users; 
* 수정
  * update users set name = 'bek' where id = 1;
* 삭제
  * delete from users where id = 1;


  

## ORM

* 데이터베이스를 객체로 추상화해 논것을 ORM(Object Relational Mapping)이라고 한다.
* 쿼리를 직접 작성하는 대신 ORM의 메소드로 데이터를 관리할 수 있는 것이 장점이다.
* 노드에서 SQL ORM은 시퀄라이져(Sequelize)가 있다.


  

## Node.js의 ORM 시퀄라이져

node의 ORM시퀄라이져를 사용하면 다음과 같이 쿼리문을 메소드형태로 바꾸어서 사용할 수 있다.

* insert users ('name') values ('alice');
  * User.create({name: 'alice'});
* select * from users;
  * User.findAll();
* update users set name = 'bek' where id = 1;
  * User.update({name: 'bek'}, {where: {id: 1}});
* delete from users where id = 1;
  * User.destory({where: {id: 1}});

메소드 앞의 User는 ORM에서 모델이라고 한다.

### 모델

* 데이터베이스 테이블을 ORM으로 추상화한것을 모델이라고 한다.

* 우리가 사용할 유저의 모델을 만들어보자

  * sequelize ORM에서 제공하는 메소드를 사용한다.
  * sequelize.define(): 모델 정의
  * sequelize.sync(): 데이터베이스 연동

* 필요 모듈 설치

  * ORM sequelize와 DB sqlite3을 설치한다.
  * `npm i sequelize sqlite3 --save`

* models.js 파일 생성

  * sequelize를 이용하여 모델을 정의한다.
  * sequelize모듈을 추출하고 new를 통하여 객체를 생성한다.
  * 이 때, 생성자로 dialect 속성으로 sqlite를 파일형태로 저장을 할 것이므로 storage속성으로 파일의 경로를 넘겨준다.
  * 다음으로 sequelize의 define을 통해서 User모델을 만든다. 속성은 name을 가지게 하고 데이터 타입은 Sequelize.STRING 타입으로 사용한다.
  * Sequelize.STRING 타입으로 생성하면 기본으로 varchar에 length 255로 기본 설정 된다.

  ```javascript
  const Sequelize = require('sequelize');
  const sequelize = new Sequelize({
    dialect: 'sqlite',
    storage: './db.sqlite'
  });

  const User = sequelize.define('User', {
    name: Sequelize.STRING
  });

  module.exports = {Sequelize, sequelize, User};
  ```




## 데이터베이스 - ORM 동기화

* 다음 순서로 데이터베이스와 위에서 생성한 ORM을 동기화한다.
* 루트의 bin 디렉토리 밑에 sync-db.js를 추가한다. 
* /bin/sync-db.js 파일내용
  * models를 추출하여 만들어진 sequelize 객체의 sync 함수를 통하여 DB와 동기화를 진행한다.
  * force: true 옵션은 이미 생성되어있는 DB를 날리고 새로 만든다는 의미이다.
  * 다음 코드를 실행하면, 루트에 db.sqlite파일이 생성 될 것이다.
  * `$ node bin/sync-db.js`
  * module.exports로 외부에서 바로 함수를 실행할 수 있도록 return한다.
  * 실행되는 models.sequelize.sync()는 내부적으로 promise를 리턴함으로써 비동기 처리를 완료할 수 있는 인터페이스를 제공한다.

```javascript
const models = require('../models');

module.exports = () => {
  return models.sequelize.sync({force: true});
};
```

* 다음 순서로 www.js 파일에서 sync-db를 실행시킨다.

  * sync() 함수가 promise 를 리턴하기 때문에, then을 사용할 수 있다.
  * 따라서, db sync가 완료되고 나서 서버를 요청 대기 상태로 바꿔준다.

  ```javascript
  const app = require('../app');
  const syncDb = require('./sync-db');

  syncDb().then(() => {
    console.log('Sync Database!');
    app.listen(3000, () => {
      console.log('Server is running on 3000 port');
    });
  });
  ```

  * npm start 결과, 아래의 로그가 찍히고 루트에는 db.sqlite파일이 생성된다.

  ```javascript
  Sync Database!
  Server is running on 3000 port
  ```


  

## API - DB 연동(index 컨트롤러 연동, findAll)

* API 로직인 user.crtl.js에서 모델을 연동하여 데이터베이스에 연결한다.

* user.ctrl.js에 선언해 높은 배열을 지우고, models를 참조하여 User객체를 불러온다.

  * `const models = require('../../models').User;`
  * 그리고 `$ npm t`를 통해서 테스트 코드를 돌려보면 User를 저장하던 배열이 사라졌으므로 모든 테스트가 실패하게 된다.
  * 우리는 이 테스트를 모두 통과시키기만 하면 된다.
  * 따라서, 하나하나씩 테스트를 실행시키기 위하여 mocha가 제공하는 옵션을 사용한다.
  * it.only() 를 사용하면 해당 테스트 케이스만 단독으로 실행한다. 이 옵션을 이용하여 하나씩 테스트 케이스를 만족시켜 갈 것이다.
  * it.only()를 통한 테스트케이스의 테스트가 끝나면,
  * describe.only()를 통하여 테스트 수트의 테스트를 진행한다.

* test코드에서 db sync를 사용하므로 user.spec.js에도 require를 통해서 models를 가져온다.

  * before메소드를 이용하여 테스트케이스가 실행되기 전에 db-sync를 해준다.
  * db에 users배열에 있는 정보를 넣어준다.(sequelize의 bulkCreate함수 사용)
  * user.spec.js

  ```javascript
  const request = require('supertest');
  const should = require('should');
  const app = require('../../app');
  const models = require('../../models');

  describe.only('GET /users는', (done) => {
    describe('성공시', () => {
      const users = [{name: 'alice'}, {name: 'bek'}, {name: 'chris'}];
      before(() => models.sequelize.sync({force: true}));
      before(() => models.User.bulkCreate(users));
      
      it('유저 객체를 담은 배열로 응답한다.', (done) => {
        request(app)
            .get('/users')
            .end((err, res) => {
              res.body.should.be.instanceOf(Array);
              done();
            });
      });

      it('최대 limit 갯수만큼 응답한다.', (done) => {
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
      
      ...
  ```

  * user.ctrl.js
  * limit을 체크하기 위해, findAll 함수의 인자로 limit을 넣어준다.
  * 그리고 then을 통하여 반환되는 promise를 처리하고, users를 받아서 json으로 응답한다.

  ```javascript
  const models = require('../../models');

  const index = function (req, res) {
    req.query.limit = req.query.limit || 10;
    const limit = parseInt(req.query.limit, 10);
    if (Number.isNaN(limit)) {
      return res.status(400).end();
    }
    models.User
        .findAll({
          limit: limit
        })
        .then(users => {
          res.json(users);
        });
  };
  	
  ...
  ```

  

## show 컨트롤러 연동(findOne)

* 현재 계속 테스트 코드를 작성하고, 테스트 실패하고, 어플리케이션 코드 수정하고, 테스트 성공하고 이 로직을 반복하고 있다.
* 이런 상황에서 계속 테스트를 수행하지 않고, mocha의 watch 옵션을 이용하면 편리하다.
* package.json의 테스트 스크립트 부분에 다음과 같이 모카의 w옵션을 준다.
* 그리고 나서 테스트를 실행시켜 보면 테스트를 수행한 이후에도 종료되지 않는다.
* 자동으로 코드의 변경 내용이 있을 경우, 테스트를 수행한다.

```javascript
... 
"scripts": {
    "test": "NODE_ENV=test mocha api/user/user.spec.js -w",
    "start": "node bin/www.js"
  },
    
    ...
```

* 테스트 코드를 user.ctrl.js의 show 부분을 검증하기 위해서, only() 함수 실행 위치를 코드상에서 변경시켜준다.
* 그러면 일부 테스트가 실패하는 것을 볼 수 있다.
* 다음과 같이 코드를 수정하여 테스트를 통과시킨다.
* user.ctrl.js코드
  * 가져온 models에 정의된 sequelize인 User의 findOne함수를 이용하여, 요청받은 id를 통해 멤버를 조회한다.
  * 이 함수 역시 promise를 리턴하므로
  * then을 통해서 리턴되는 user객체를 가져와서, user가 없다면 404 코드를 리턴하고,
  * user가 존재한다면 user를 리턴하도록 코드를 수정한다.
  * 터미널 창에 가보면 바뀐 코드를 수행하여, 테스트에 통과된 결과를 확인 할 수 있을 것이다.

```javascript
const models = require('../../models');

...

const show = function (req, res) {
  const id = parseInt(req.params.id);
  if (Number.isNaN(id))
    return res.status(400).end();

  models.User
      .findOne({
        where: {id}
      })
      .then(user => {
        if (!user) return res.status(404).end();
        res.json(user);
      });
};

...
```

  

