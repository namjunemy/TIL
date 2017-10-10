# NPM(Node Package Manager)

이전의 학습에서 npm install을 통해 express와 morgan 모듈을 설치했는데, node_modules에 가보면 다른 기본적인 많은 모듈들도 같이 설치 되어있다.

### package.json

* 우리가 git을 통해서 코드 관리를 할때, js파일을 주로 관리하지 node_modules까지 같이 push하진 않는다.
* 따라서, git repo를 clone을 받아서 사용할 때 모듈 설치 없이 사용할 수 없다.
* `$ npm init` 을 통해서 package.json을 생성한다.
* package.json의 내용을 보면, 의존 정보에 관한 내용들이 있다.
* 또한, 우리가 설치했던 express와 morgan에 관한 의존정보들도 함께 관리를 해준다.
* 이렇게 package.json에 설치한 모듈의 의존정보를 관리하려면,
* `$ npm install express --save` 이렇게 설치를 해준다.
* 이렇게 package.json이 관리가 되면, git clone 후에 또는 node_modules가 존재하지 않을 경우
* `$ npm install`을 이용하여 간편하게 의존 모듈들을 설치를 할 수 있다.

  

### package.json -> scripts

* package.json의 scripts에 start를 추가하면
* `$ npm start`로 실행할 수 있다.

```javascript
{
  "name": "hello_express",
  "version": "1.0.0",
  "description": "",
  "main": "helloexpress.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
    "start": "node index.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.16.2",
    "morgan": "^1.9.0"
  }
}

```

