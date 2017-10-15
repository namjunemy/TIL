## 코드 리팩토링

* 지금까지 우리가 개발한 app.js에는 모든 어플리케이션 코드가 들어가 있다.
* 이 어플리케이션 코드를 테스트 하는 부분이 app.spec.js에 구현되어 있다.
* api가 추가 될수록, app.js파일이 수도 없이 늘어날 수 있다.
* 따라서, 역할에 따라서 파일을 분리한다.

### 파일 구조

* 구조

  * app.js
  * api/
    * user/
      * index.js - 기본적인 라우팅 설정 로직
      * user.ctrl.js - 실제 API의 로직
      * user.spec.js - 테스트 코드

* api/user/index.js 생성, app.js 파일 변경

  * 이전에 express를 학습하면서, 라우팅이라는 개념이 있었다.
  * 이 때, 라우팅을 위한 전용 라우터 클래스를 사용할 수 있다. 여기서 사용한다.
  * express.Router 클래스를 사용하면 모듈식 마운팅 가능한 핸들러를 작성할 수 있다.
  * 변경된 app.js 파일
    * 변경내용은 다음과 같다
    * api/user/index.js 를 추출하여 모듈형태로 가져와서, 미들웨어에 추가한다.
    * /users로 들어오는 모든 API 요청은 user 모듈이 담당한다는 의미이다.

  ```javascript
  const express = require('express');
  const app = express();
  const morgan = require('morgan');
  const bodyParser = require('body-parser');
  const user = require('./api/user/index');

  app.use(morgan('dev'));
  app.use(bodyParser.json());
  app.use(bodyParser.urlencoded({extended: true}));

  app.use('/users', user);

  app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
  });

  module.exports = app;
  ```

  * 추가된 api/user/index.js 파일
    * 변경내용은 다음과 같다.
    * app.js의 라우팅 관련 코드와 배열을 모두 떼어다가 api/user/index로 옮겼다.
    * 그리고 express.모듈을 추출하고 Router클래스를 사용할 준비를 한다.
    * 이전에 app.get() 메소드를 사용하던 것을 router객체의 get, post, delete, put메소드로 대체 할 수 있다.
    * 또한, app.js에서 index를 미들웨어 형태로 추가할 때 '/users'로 들어오는 모든 요청을 담당하게 했으므로, 해당 메소드에서 요청할때 앞에 '/users'경로를 제거한다. 
    * 그리고 module.exports를 통해 app.js에서 router 객체를 사용할 수 있도록 한다.

  ```javascript
  const express = require('express');
  const router = express.Router();

  let users = [
    {id: 1, name: 'alice'},
    {id: 2, name: 'bek'},
    {id: 3, name: 'chris'}
  ];

  router.get('/', (req, res) => {
    req.query.limit = req.query.limit || 10;
    const limit = parseInt(req.query.limit, 10);
    if (Number.isNaN(limit)) {
      return res.status(400).end();
    }
    res.json(users.slice(0, limit));
  });

  router.get('/:id', (req, res) => {
    const id = parseInt(req.params.id);
    if (Number.isNaN(id))
      return res.status(400).end();

    const user = users.filter((user) => user.id === id)[0];
    if (!user)
      return res.status(404).end();

    res.json(user);
  });

  router.delete('/:id', (req, res) => {
    const id = parseInt(req.params.id);
    if (Number.isNaN(id))
      return res.status(400).end();
    users = users.filter(user => user.id !== id);
    res.status(204).end();
  });

  router.post('/', (req, res) => {
    const name = req.body.name;
    if (!name)
      return res.status(400).end();

    const isConflict = users.filter(user => user.name === name).length;
    if (isConflict)
      return res.status(409).end();

    const id = Date.now() + 1;
    const user = {id, name};
    users.push(user);
    res.status(201).json(user);
  });

  router.put('/:id', (req, res) => {
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

  module.exports = router;
  ```

* api/user/user.ctrl.js 분리하기

  * index.js에서는 라우팅에 대한 컨트롤러 함수를 binding 역할을 하도록 하고, user.ctrl.js에서는 컨트롤러 로직에 대해서만 정의를 한다.
  * 변경된 index.js

  ```javascript
  const express = require('express');
  const router = express.Router();
  const ctrl = require('./user.ctrl');

  router.get('/', ctrl.index);
  router.get('/:id', ctrl.show);
  router.delete('/:id', ctrl.destroy);
  router.post('/', ctrl.create);
  router.put('/:id', ctrl.update);

  module.exports = router;
  ```

  * 추가된 user.ctrl.js

  ```javascript
  let users = [
    {id: 1, name: 'alice'},
    {id: 2, name: 'bek'},
    {id: 3, name: 'chris'}
  ];

  const index = function (req, res) {
    req.query.limit = req.query.limit || 10;
    const limit = parseInt(req.query.limit, 10);
    if (Number.isNaN(limit)) {
      return res.status(400).end();
    }
    res.json(users.slice(0, limit));
  };

  const show = function (req, res) {
    const id = parseInt(req.params.id);
    if (Number.isNaN(id))
      return res.status(400).end();

    const user = users.filter((user) => user.id === id)[0];
    if (!user)
      return res.status(404).end();

    res.json(user);
  };

  const destroy = function (req, res) {
    const id = parseInt(req.params.id);
    if (Number.isNaN(id))
      return res.status(400).end();
    users = users.filter(user => user.id !== id);
    res.status(204).end();
  };

  const create = function (req, res) {
    const name = req.body.name;
    if (!name)
      return res.status(400).end();

    const isConflict = users.filter(user => user.name === name).length;
    if (isConflict)
      return res.status(409).end();

    const id = Date.now() + 1;
    const user = {id, name};
    users.push(user);
    res.status(201).json(user);
  };

  const update = function (req, res) {
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
  };

  module.exports = {index, show, destroy, create, update};
  ```

* api/user/user.spec.js 파일 분리하기

  * 이 파일은 app.spec.js 파일을 그대로 가져온다.
  * 하나, app.js의 상대 경로가 바뀌었으므로 이 부분만 주의해서 변경한다.

  ```javascript
  const request = require('supertest');
  const should = require('should');
  const app = require('../../app');

  describe('GET /users는', (done) => {
    describe('성공시', () => {
      it('유저 객체를 담은 배열로 응답한다.', () => {
      ...
      
  ...(이하 생략)
  ```

* package.json의 테스트 스크립트의 경로도 수정을 해주고, 테스트를 해보면 정상적으로 수행된다.

  

## 테스트 환경 개선

* 테스트 결과를 보는 로그 중간 중간, 어플리케이션에 관한 로그 등 불필요한 로그가 같이 찍힌다.

* 테스트 결과는 테스트에 관한 내용만 보여주는 것이 좋다.

* 따라서, 환경 변수를 사용해서 test의 결과를 깔끔하게 확인할 수 있도록 한다.

  * 변경된 package.json의 test 스크립트
    * NODE_ENV를 test로 할당해서 모카를 실행한다.

  ```javascript
  {
    "name": "tdd_api_server",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
      "test": "NODE_ENV=test mocha api/user/user.spec.js",
      "start": "node bin/www.js"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
      "body-parser": "^1.18.2",
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

  * 변경된 app.js 파일
    * process.env.NODE_ENV가 test일 경우에만 서버의 로그가 찍히도록 했다.
    * 그리고 테스트 코드에서 supertest객체가 express 서버를 중복으로 구동하므로, app.js에서 app.listen() 메소드를 제거한다.
    * 그리고 별도의 파일로 분리하여 npm start로 어플리케이션 코드를 실행 했을 때 문제가 없도록 한다.

  ```javascript
  const express = require('express');
  const app = express();
  const morgan = require('morgan');
  const bodyParser = require('body-parser');
  const user = require('./api/user/index');

  if(process.env.NODE_ENV !== 'test') {
    app.use(morgan('dev'));
  }

  app.use(bodyParser.json());
  app.use(bodyParser.urlencoded({extended: true}));

  app.use('/users', user);

  module.exports = app;
  ```

  * /bin/www.js 파일 추가

    * 프로젝트 루트에 bin 폴더를 만들고 www.js에 아래의 내용을 추가한다.
    * package.json의 npm start 경로를 bin/www.js로 변경한다.
    * npm start로 서버를 구동해도 문제가 없어졌다.

    ```javascript
    const app = require('../app');

    app.listen(3000, () => {
      console.log('Server is running on 3000 port');
    });
    ```

* 테스트 환경에서 로그를 깔끔하게 볼 수 있도록 코드를 수정함과 동시에 실제 어플리케이션 구동에도 문제가 없도록 코드를 변경했다.