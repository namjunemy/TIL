

# 쉽게 배우는 Webpack

> 인프런 - CAPTAIN PANGYO([강좌링크](https://www.inflearn.com/course/webpack-%EC%9B%B9%ED%8C%A9-%EA%B0%95%EC%A2%8C/))

## 개요

* React, Angular, Vue 에서 권고하는 Webpack 설정에 대해 학습 및 이해
* Webpack의 주요 설정 Entry, Output, Loader, Plugins, Resolve 학습 및 실습
* 실무에 적용할 수 있는 Webpack 개발환경 설정방법 학습 및 실습

## Webpack 이란?

* 서로 연관 관계가 있는 웹 자원들을 js, css, img와 같은 static한 자원으로 변환해주는 모듈 번들러
* 아래 사진은 직관적으로 webpack의 역할을 설명
  * 왼쪽에 보이는 .js, .jade, .css등 웹 페이지를 동작시키기 위해 서로 관계를 맺고있는 웹 어플리케이션에 관련된 파일들을 웹팩에서 인식해서 번들링 하게 되면, 
  * 최종적으로 기존의 파일들을 압축하고 축소하게 된다.
  * 정리하자면, 웹 페이지를 구성할 때 필요한 파일들의 관계를 웹팩에서 인식하고, 웹 페이지에 올라가기 직전에 최적화된 파일 set으로 뽑아내는게 웹팩의 역할이다.
    * 두가지 정도 참고할 사항은
    * 웹팩은 모듈간의 관계들을 정리한 의존성 그래프를 갖게되므로 모듈관리를 수월하게 할 수 있고
    * 자원들을 최적화해서 압축하므로 웹페이지의 성능을 끌어올리는 역할을 한다.

![](https://github.com/namjunemy/TIL/blob/master/Webpack/img/webpack01.PNG?raw=true)

## Webpack을 사용하는 이유 & 배경

1. 새로운 형태의 Web Task Manager
   * 기존 Web Task Manager(Gulp, Grunt)의 기능 + 모듈 의존성 관리
   * 예) minification(웹 자원을 압축하고 성능을 향상시키기 위한 기법)을 webpack default cli로 실행 가능
   * `webpack -p`
2. 자바스크립트 Code based Modules 관리
   * 자바스크립트의 모듈화 필요성 : AMD(비동기 모듈 정의), Common js, ES6(Modules)
     * 자바스크립트 언어 특성(전역 변수 스코프), 기타 프로그래밍 언어에 비해서 파일 base로 모듈화가 되지 않는다.
   * 기존 모듈 로더들과의 차이점 : 모듈 간의 관계를 청크(chunk) 단위로 나눠 필요할 때 로딩
   * 현대의 웹에서 JS 역할이 커짐에 따라, Client Side에 들어가는 코드량이 많아지고 복잡해짐
   * 복잡한 웹 앱을 관리하기 위해 모듈 단위로 관리하는 Common JS, AMD, ES6 Modules 등이 등장
   * 가독성이나 다수 모듈 미병행 처리등의 약점을 보완하기 위해 Webpack이 등장

### 자바스크립트 모듈화 문제란?

* 아래는 script 태그로 간단하게 자바스크립트 모듈화를 시도하는 예제

```js
<script src="module1.js"></script>
<script src="module2.js"></script>
<script src="lib1.js"></script>
<script src="module3.js"></script>
```

* 위의 모듈 로딩 방식의 문제점 : 전역변수 충돌, 스크립트 로딩 순서, 복잡도에 따른 관리상의 문제
* 이를 해결하기 위해 AMD 및 기타 모듈 로더들, Webpack이 등장

## Webpack의 철학

1. Everything is Module

   * 모든 웹 자원(js, css, html)이 모듈 형태로 로딩 가능

   ```js
   require('base.css');
   require('min.js');
   ```

2. Load only "what" you need and "when" you need
   * 초기에 불필요한 것들을 모두 로딩하지 않고, 필요할 때 필요한 것만 로딩하여 사용

## Webpack 개발환경 설정

> Atom, Chrome, Node.js, NPM, github 설치

* webpack cli 설치
  * 버전이 올라가면서 `npm i webpack-cli g`로 webpack-cli를 설치해야 한다.

## Webpack 시작하기

간단한 webpack 기본실습

[실습 가이드 링크](https://github.com/joshua1988/LearnWebpack)

#### 실습절차

1. webpack 전역 설치
2. `npm init` 으로 package.json 생성
3. app/index.js 와 index.html 생성
4. js와 html에 코드 추가
5. `webpack app/index.js dist/bundle.js` 명령어 실행후 index.html 로딩
   * 웹팩 버전업 후 변경
   * `webpack app/index.js --output dist/bundle.js --mode=development`
6. webpack.config.js 파일 추가 후 webpack
   * 버전업 후 `--mode=development` 붙여야 에러 안남

### 번들링된 파일 bundle.js 분석

* 대상 파일인 index.js 파일의 내용이 들어가 있고,
* lodash 라이브러리 관련 내용들이 정책에 의해서 파일의 최상단에 @license와 함께 포함된다.
* 웹팩에서 여러개의 자바스크립트 파일을 하나로 번들링하게 되므로 파일의 갯수가 줄어들고, 웹팩의 production mode를 사용하면 압축까지 되면서 성능상의 이점을 가져올 수 있다. 



