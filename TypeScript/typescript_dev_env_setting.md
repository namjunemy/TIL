# 개발환경 구축 및 컴파일러 사용

> '인프런 - 타입스크립트 코리아: 기초 세미나'를 학습한 내용입니다.
>
> OS: macOS Sierra



## 자바스크립트 실행환경 설치

### node.js

크롬의 v8 자바스크립트 엔진을 사용하여 자바스크립트를 해석하고, OS 레벨에서의 API를 제공하는 서버사이드용 자바스크립트 런타임환경.

* brew install node
* brew insatll nvm
  * node version manager
  * nodejs의 버전을 관리해주는 매니저



## 타입스크립트 컴파일러 설치

* npm으로 설치

  * global

    * npm i typescript -g
    * node_modules/.bin/tsc
    * tsc source.ts

  * local

    * 타입스크립트 프로젝트 디렉토리 생성 후 이동

    * 해당 위치에서 아래 작업 진행

    * npm init

    * npm i typesctipt

    * 그러면 node_modules에 typescript가 생성됨.

    * node_modules에 있는 .bin안에 있는 명령은 npm script에서는 앞에 경로 안쓰고 쓸 수 있다.

    * 따라서, package.json을 다음과 같이 수정할 수 있다. "transpile": "tsc"

      ~~~json
      {
        "name": "typescript-basic",
        "version": "1.0.0",
        "description": "",
        "main": "index.js",
        "scripts": {
      	"transpile": "tsc test.ts",
          "test": "echo \"Error: no test specified\" && exit 1"
        },
        "author": "",
        "license": "ISC",
        "dependencies": {
          "typescript": "^2.5.2"
        }
      }
      ~~~

    * npm run transfile 또는 ./node_modules/.bin/tsc 로 실행

      ~~~typescript
      class Test {
        constructor() {
      	  console.log('test');
        }
      }

      new Test();
      ~~~

    * tsc —init 을 통해서 tsconfig.json을 생성하면, tsconfig.json의 설정대로

    * tsc 명령어를 통해서 컴파일 할 수 있다.





# IDE 활용(IntelliJ Idea)



- 앞서서 테스트한 project 디렉토리 열고,

- Preference -> Languages & Frameworks -> TypeSctript 이동

- Node interpreter 설정

- TypeScript version -> detect automatically 선택

- TSLint -> Enable 체크 -> tslint를 자동으로 찾아줌.

  ​