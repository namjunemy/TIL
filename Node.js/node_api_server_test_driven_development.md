# TDD(Test Driven Development)

## 테스트 주도 개발이란?

* 개발을 할 때, 바로 소스코드를 작성하지 않고 테스트 코드를 먼저 만드는 것


* 테스트를 하나하나 통과해 나가면서 코드를 만들어 나가는 방법
* 물론 TDD를 하다 보면 개발에 시간이 많이 걸리는 것은 사실이다
* 그러나, 유지보수하는 시점에 가보면 TDD로 개발했던 것들이 유지보수 시간을 줄이는데 엄청난 도움이 된다!
* node에서 TDD를 위해 3가지 라이브러리를 사용한다.
  * Mocha, should, superTest

  

## mocha

* 모카는 테스트 코드를 실행시켜주는 역할을 한다. 그래서 보통 테스트 러너라고 한다.

* [https://mochajs,org](https://mochajs,org) 홈페이지로 이동하면, 모카를 사용하는 방법까지 쭉 설명 되어 있다.

* 모카는 두가지 파트로 나뉘어져 있다.

  * 테스트 수트 : 테스트 환경으로 모카에서는 describe()라는 함수로 구현한다.
  * 테스트 케이스 : 테스트 수트와는 다르게 실제 테스트를 만드는 역할을 하고, 모카에서는 it()이라는 함수로 구현한다.

* 모카를 이용해서 테스트 케이스 샘플 작성하기

  * utils.js 생성

    ```javascript
    function capitalize(str) {
      return str;
    }

    module.exports = {capitalize: capitalize};
    ```

  * utils.spec.js 생성

    * utils 모듈을 가져오고 테스트 할 준비를 한다.
    * describe() 함수를 통해서 테스트 수트(테스트 환경)을 만든다.
      * 여기서는 간단히 테스트할 대상에 대해 서술식으로 써주면 된다.
    * it() 함수를 통해 실제 테스트 할 테스트 케이스를 만들고 내용을 입력한다.
      * 어떤 테스트 케이스 인지 설명할 문자열을 넣어준다.
    * 실제 테스트 코드의 내용이 맞는지 검증할 기본 모듈인 assert 모듈을 사용한다.

    ```javascript
    const utils = require('./utils');
    const assert = require('assert');

    describe('utils.js모듈의 capitalize() 함수는', () => {
      it('문자열의 첫번쨰 문자를 대문자로 변환한다', () => {
        const result = utils.capitalize('hello');
        assert.equal(result, 'Hello');
      });
    });
    ```

    * 이 코드는 node가 바로 실행 시킬 수 없다. mocha가 테스트 코드를 돌려주는 역할을 한다!

    * 모카를 설치한다

      * dev 옵션을 주어서 설치하는 이유는 개발환경에서 필요한 모듈이라는 의미이다.
      * `$ npm install mocha --save-dev`
      * 이렇게 설치를 하면 save옵션을 줬으므로, package.json으로 가보면 devDependencies 항목 안에 추가가 되어있는 것을 볼 수 있다.

    * 테스트 코드를 실행한다.

      * 모카의 실행파일은 node_modules/.bin/mocha 위치에 설치 되어 있다.
      * `$ node_modules/.bin/mocha utils.spec.js` 명령을 입력하여 실행한다.
      * 실행결과는 다음과 같다. 
      * '1) 문자열의 첫번쨰 문자를 대문자로 변환한다' 이 부분이 빨간줄로 출력되면서 실패했다는 결과를 얻을 수 있다.

      ```shell
        utils.js모듈의 capitalize() 함수는
          1) 문자열의 첫번쨰 문자를 대문자로 변환한다


        0 passing (9ms)
        1 failing

        1) utils.js모듈의 capitalize() 함수는
             문자열의 첫번쨰 문자를 대문자로 변환한다:

            AssertionError [ERR_ASSERTION]: 'hello' == 'Hello'
            + expected - actual

            -hello
            +Hello
            
            at Context.it (utils.spec.js:7:12)
      ```

      * utils.js의 capitalize() 함수를 수정해서 테스트를 통과하도록 한다.

      ```javascript
      function capitalize(str) {
        return str.charAt(0).toUpperCase() + str.slice(1);
      }

      module.exports = {capitalize: capitalize};
      ```

      * `$ node_modules/.bin/mocha utils.spec.js` 명령을 입력하여 다시 테스트를 실행한다.
      * 결과는 다음과 같다. 테스트에 성공했다.

      ```shell
       utils.js모듈의 capitalize() 함수는
          ✓ 문자열의 첫번쨰 문자를 대문자로 변환한다


        1 passing (9ms)
      ```

      * 이렇게 describe() 함수와 it() 함수로 테스트코드를 작성하고, mocha를 통해 테스트코드를 실행할 수 있다.

  

## Should

* 노드 공식 문서의 assert 항목에 가보면, assert모듈은 테스트 코드에서는 사용하지 말라고 얘기한다.

* 테스트 코드의 검증에는 서드 파티 라이브러리를 사용하라고 얘기한다.

* 그래서 실제 테스트 코드에서 검증에 사용되는 것이 Should 라는 라이브러리이다.

* [https://shouldjs.github.org](https://shouldjs.github.org) 공식홈페이지 또는, [https://gitjub.com/tj/should.js/](https://gitjub.com/tj/should.js/) 깃헙에 가보면 더 자세한 정보를 얻을 수 있다.

* should를 사용하기 위해 설치를 진행한다.

  * `$ npm install should --save-dev`

* 그렇다면, 위에서 assert로 작성된 검증 코드를 should로 바꿔본다.

  * utils.spec.js 파일

  ```
  const utils = require('./utils');
  const should = require('should');

  describe('utils.js모듈의 capitalize() 함수는', () => {
    it('문자열의 첫번쨰 문자를 대문자로 변환한다', () => {
      const result = utils.capitalize('hello');
      result.should.be.equal('Hello');
    });
  });
  ```

  * should 를 사용하여 테스트 코드를 작성 한 결과 조금 더 영어 문장을 읽는 듯한 느낌으로 테스트 코드의 가독성을 좋게 한다.
  * 실행결과는 같은 결과를 얻는다.
  * should 라이브러리를 통해서 테스트 코드의 가독성을 높일 수 있다!!

  

## SuperTest

* 세번째로 사용하는 라이브러리로 SuperTest가 있다.

* 지금까지 위에서 진행한 테스트는 테스트 종류로 보자면 단위 테스트이다. 즉, 함수의 기능을 테스트 한 것이다.

* SuperTest는 통합테스트에 사용 된다. 이 학습에서는 최종 결과물인 API에 대한 기능 테스트를 말한다.

* 규모로 보자면 단위 테스트보다 큰 테스트이다.

* SuperTest는 Express 통합 테스트용 라이브러리이다.

  * 그래서 SuperTest는 내부적으로 Express 서버를 구동시키고, 실제 테스트 코드에서 작성된 시나리오 대로 요청을 보낸다.
  * 그 요청에 대한 결과를 가지고 검증을 하는 역할을 한다.

* [https://github.com/visionmedia/supertest](https://github.com/visionmedia/supertest) 깃허브 링크를 참조하면 다음과 같은 예제 코드를 볼 수 있다.

  * supertest 모듈을 추출해서 request라는 변수로 사용한다.
  * request()함수에서 express 객체를 넣어주면, supertest는 내부적으로 express를 구동한다.
  * 그런 다음에 실제로 요청 시나리오를 만든다.
    * GET 메소드의 /users 경로를 가지고 있는 request를 실제 express서버에 요청하고
  * 그 다음으로 expect()함수를 통해서 차례차례 검증을 시작한다.
    * expect('Content-Type', /json/) : 응답헤더의 컨텐츠 타입은 json이어야 하며
    * expect('Content-Length', '15')  : 컨텐츠의 길이는 15이고
    * expect(200) : 받는 상태코드는 200 이어야 한다.
    * 라고 테스트 코드를 작성한다.
    * 그리고 마지막으로 end()를 통해 응답을 받고, err가 발생하게 되면 err를 던진다.

  ```javascript
  const request = require('supertest');
  const express = require('express');

  const app = express();

  app.get('/user', function(req, res) {
    res.status(200).json({ name: 'tobi' });
  });

  request(app)
    .get('/user')
    .expect('Content-Type', /json/)
    .expect('Content-Length', '15')
    .expect(200)
    .end(function(err, res) {
      if (err) throw err;
    });
  ```

* 그렇다면, 다음 순서로 처음에 만들었던 api를 테스트 한다.

* 기존에 만들었던 app.js 파일

  * 테스트 코드 에서 app 객체를 사용하기 위해 변수를 직접 exports 했다.

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

  modules.exports = app;
  ```

* 이 코드를 테스트 하기 위한 app.spec.js 파일 생성

  * 마찬가지로 describe 함수를 통해서 테스트 수트를 만들고,
  * it 함수를 통해 테스트 케이스를 작성한다.
  * 그 다음, supertest 객체의 변수인 request를 통하여 express 객체를 넣어주고
  * 실제로 express 서버에 /users 경로로 get요청을 보내기 위한 코드를 작성한다.
  * 결과값이 제대로 넘어오는지 확인하기 위해서 end() 함수를 통해 출력한다.
  * 이 때, 주의해야 할 점은 위에서 만든 app.js파일의 API는 비동기로 동작하게 되어있기 때문에, 비동기 처리를 해줘야 한다. 아주 간단하게 할 수 있다.
  * it 함수의 두번째 파라미터로 함수를 넣어줄 수 있는데,  done이라는 콜백함수를 넣어준다.
  * 그리고 테스트가 끝나는 시점에서 done() 함수를 호출해 주기만 하면 된다.

  ```javascript
  const app = require('./app');
  const request = require('supertest');

  describe('GET /users는', () => {
    it('슈퍼테스트 실행', () => {
      request(app)
          .get('/users')
          .end((err, res) => {
            console.log(res.body);
            done();
          });
    });
  });
  ```

  * 모카를 통해서 실행을 시켜보면, 아래와 같이 express서버가 구동되고 테스트를 진행하는 결과를 볼 수 있다.

  ```shell
  Example app listening on port 3000!
    GET /users는
      ✓ 슈퍼테스트 실행
  GET /users 200 3.563 ms - 71
  [ { id: 1, name: 'alice' },
    { id: 2, name: 'bek' },
    { id: 3, name: 'chris' } ]


    1 passing (37ms)

  ```

  ​