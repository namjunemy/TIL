# Webpack을 위한 NPM 소개

## NPM? (Node Package Manager)

* js 개발자들이 편하게 개발할 수 있도록 js 라이브러리들을 모아놓은 열린 공간
* front-end web apps, mobile apps, robots, routers 라이브러리들이 돈재
* Gulp, Webpack 모두 node 기반, NPM을 사용하여 필요 라이브러리들을 로딩
* 재사용 가능한 code를 module, package라고 호칭
* package.json : 해당 package에 대한 파일 정보가 들어가 있음
* keyword로 패키지 검색 가능
* [NPM 공식 사이트](https://www.npmjs.com/)

## NPM 명령어 & package.json 파일

* Package.json 생성
  * 프로젝트의 node module들 관리를 위한 package.json 설정
  * `-y` 를 붙이면 default 정보를 가지고 package.json을 생성

```shell
npm init
npm init -y
```

* 패키지 설치 및 업데이트
  * 프로젝트에서 사용할 node module 설치 및 package.json 업데이트

```shell
npm install 패키지명 --save
npm i 패키지명 --save
```

* install --save와 --save-dev
  * `--save`는 앱이 구동하기 위해 필요한 모듈 & 라이브러리 설치. ex) react, vue

  ```js
  "dependencies": {
    "vue": "^2.3.3"
      ...
  },
  ```

  * `--save-dev`는 앱 개발시에 필요한 모듈 & 라이브러리 설치. ex) test, build tool, live reloading