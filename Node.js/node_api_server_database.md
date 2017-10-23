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

  

## destroy 컨트롤러 연동(delete)

* 같은 방법으로 delete에 해당하는 destroy 컨트롤러와 연동한다.
* user.ctrl.js 코드 수정
  * destroy()함수의 where 인자로 id를 넣어주고, then을 통해 넘어오는 결과로 204를 리턴한다.

```javascript
...

const destroy = function (req, res) {
  const id = parseInt(req.params.id);
  if (Number.isNaN(id))
    return res.status(400).end();

  models.User
      .destroy({
        where: {id}
      })
      .then(() => {
        res.status(204).end();
      });
};

...
```

* user.spec.js 코드

```javascript
describe.only('DELETE /users/1은', () => {
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

  

## create 컨트롤러 연동

* 계속해서 only() 함수 위치를 옮겨가며 테스트를 진행한다.
* user.ctrl.js 코드
  * models의 User의 create함수를 이용하여 name을 넘겨준다.
  * then을 통해 생성된 user를 리턴받으면 201을 리턴한다.
  * 만약 err가 발생되고, 그것이 'SequelizeUniqueConstraintError'라는 unique하지 못함을 뜻하는 에러라면 409를 리턴하고, 그것이 아닐경우 500을 리턴하도록 코드를 변경한다.

```javascript
...
const create = function (req, res) {
  const name = req.body.name;
  if (!name)
    return res.status(400).end();
  
  models.User
      .create({name})
      .then(user => {
        res.status(201).json(user);
      })
      .catch(err => {
        if (err.name === 'SequelizeUniqueConstraintError') {
          return res.status(409).end();
        }
        return res.status(500).end();
      });
};
...
```

* 'SequelizeUniqueConstraintError' unique의 여부를 판단하기 위해서 sequelize의 설정을 다음과 같이 해준다.
  * models.js코드
    * ORM 모델을 정의할 때, name의 속성 값중 unique속성을 true로 준다.

```javascript
const Sequelize = require('sequelize');
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: './db.sqlite',
  logging: false
});

const User = sequelize.define('User', {
  name: {
    type: Sequelize.STRING,
    unique: true
  }
});

module.exports = {Sequelize, sequelize, User};
```

* user.spec.js

```javascript
...
describe.only('POST /users는', () => {
  const users = [{name: 'alice'}, {name: 'bek'}, {name: 'chris'}];
  before(() => models.sequelize.sync({force: true}));
  before(() => models.User.bulkCreate(users));
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
...
```

  

## update 컨트롤러 연동

* user.ctrl.js 코드
  * 요청에 대한 400 응답 처리 코드는 기존과 동일.
  * User의 findOne 메소드를 통해서 해당 id를 가진 사용자가 존재하는지 먼저 확인한다.
  * then으로 넘어오는 리턴값이 없다면 404를 리턴
  * 그리고 user가 넘어 왔다면, user.name을 요청한 name으로 저장하고
  * save 함수를 통해서 db에 저장한다. 성공시 넘어오는 값을 json으로 리턴하고,
  * create와 마찬가지로 중복에러가 발생할경우 409를, 그외의 에러일 경우 500을 리턴한다. 

```javascript
...
const update = function (req, res) {
  const id = parseInt(req.params.id, 10);
  if (Number.isNaN(id))
    return res.status(400).end();

  const name = req.body.name;
  if (!name)
    return res.status(400).end();

  models.User
      .findOne({where: {id}})
      .then(user => {
        if (!user) return res.status(404).end();

        user.name = name;
        user.save()
            .then(() => {
              res.json(user);
            })
            .catch(err => {
              if (err.name === 'SequelizeUniqueConstraintError') {
                return res.status(409).end();
              }
              return res.status(500).end();
            })
      });
};
...
```

* user.spec.js 코드

```javascript
...
describe('PUT /users/:id', () => {
  const users = [{name: 'alice'}, {name: 'bek'}, {name: 'chris'}];
  before(() => models.sequelize.sync({force: true}));
  before(() => models.User.bulkCreate(users));
  describe('성공시', () => {
    it('변경된 name을 응답한다', (done) => {
      const name = 'chally';
      request(app)
          .put('/users/3')
          .send({name})
          .end((err, res) => {
            res.body.should.have.property('name', name);
            done();
          });
    });
  });
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
});
...
```



## 마무리

* 데이터베이스와 컨트롤러 연동을 마쳤고, 데이터가 잘 유지되는지 보기 위한 확인을 진행한다.

* 우선 db.sqlite파일을 지우고, curl 명령이 아닌 postman을 사용하여 확인을 진행한다.

* 진행하기 전에, db-sync.js에서 db를 sync하는 부분의 force 옵션을 실제 구동 환경에서는 false로, 테스트 환경에서는 true로 하도록 코드를 수정한다.

  * db-sync.js

  ```javascript
  const models = require('../models');

  module.exports = () => {
    const options = {
      force: process.env.NODE_ENV === 'test' ? true : false
    };
    return models.sequelize.sync({options});
  };
  ```

* 각 요청에 대한 응답코드와 결과가 잘 돌아오는지 확인한다.

* 또한, 서버 쪽에서 찍히는 로그도 잘 찍히는지 확인한다.

* 추가적인 api를 생성할 때는 api폴더의 하위 폴더를 생성하고, app.js에 라우팅 하는 부분에 추가만 해주면 된다.

* 물론, 어떤 api를 추가하건 테스트 코드를 먼저 만들고, 실제 비즈니스 로직을 만들어야 함을 유념해야한다.

* 끝으로 정말 유익하고 재미있는 강의를 해주신 우아한형제들의 김정환님께 감사하고, tdd에 입문하는 분들께 이 강의를 적극 추천한다.

* 강의 링크

  * [https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api/](https://www.inflearn.com/course/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%A3%BC%EB%8F%84%EA%B0%9C%EB%B0%9C-tdd-nodejs-api/) 