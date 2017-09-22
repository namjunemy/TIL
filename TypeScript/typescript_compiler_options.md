# Compiler Options

>  '인프런 - 타입스크립트 코리아: 기초 세미나'를 학습한 내용입니다.





## 타입스크립트 컴파일러 옵션

* [http://json.schemastore.org/tsconfig](http://json.schemastore.org/tsconfig) 여기에 정확히 나와있다.



## 최상위 프로퍼티

* compileOnSave
  * true / false(default false)
  * 이벤트를 받는 특정 IDE가 필요하다.
    * VS 2015 with TypeScript(1.8.4 이상)
    * Atom-typescript 플러그인
* extends
  * 파일 (상대) 경로명: string
  * Typesctipt 2.1 New Spec 
* files, include, exclude
  * 셋다 설정이 없으면, 전부다 컴파일
  * files
    * 상대 혹은 절대 경로의 리스트 배열입니다.
    * exclude 보다 쎄다.
  * include, exclude
    * glob 패턴(마치 .gitignore 같이)
    * include
      * exclude 보다 약하다.
      * *을 사용하면, .ts/.tsx/.d.ts 만 include (allowJS)
    * exclude
      * 설정 안하면 4가지(node_modules, bower_components, jspm_packages, \<outDir\>)를 default로 제외한다.
      * \<outDir\>은 항상 제외한다.(include에 있어도)
* compileOptions
  * 옵션이 굉장히 많다. 필요한 것 찾아서 쓰면 된다. 제일 중요한 것 위주로 몇개만 본다.
  * type
    * @types
      * TypeScript 2.0부터 사용 가능해진 내장 type definition 시스템
        * 아무 설정을 안하면?
          * Node_modules/@types 라는 모든 경로를 찾아서 사용
        * typeRoots를 사용하면?
          * 배열 안에 들어있는 경로들 아래서만 가져온다.
        * types를 사용하면?
          * 배열 안의 모듈 혹은 ./node_modules/@types/ 안의 모듈 이름에서 찾아온다.
          * [] 빈 배열을 넣는다는건 이 시스템을 이용하지 않겠다는 것이다.
        * typeRoots와 types를 같이 사용하지 않는다.
  * target
    * 빌드의 결과물을 어떤 버전으로 할 것이냐
    * 놀랍게도 default는 es3
  * lib
    * 기본 type definition 라이브러리를 어떤 것을 사용할 것이냐
    * lib를 지정하지 않을 때,
      * terget이 'es3'이고, 디폴트로 lib.d.ts를 사용한다.
      * terget이 'es5'이면, 디폴트로 dom, es5, scripthost를 사용한다.
      * terget이 'es6'이고, 디폴트로 dom, es6, dom.iterable, scripthost를 사용한다.
    * lib를 지정하면 그 lib 배열로만 라이브러리를 사용한다.
      * 빈 [] 설정하면 -> 'no definition found ….'
  * outDir, outFile
    * outFile
      * 프로젝트를 컴파일 된 하나의 파일로 내보낼 때.
    * outDir
      * 소스 디렉토리를 구조 그대로 컴파일 된 상태로 옮길 때.
  * module
    * 모듈시스템을 말한다. commonjs, amd, system 등
    * 컴파일 된 모듈의 결과물을 어떤 모듈 시스템으로 할지를 결정
    * target이 es6이면 es6가 디폴트이고,
    * target이 ex6가 아니면 commonjs가 디폴트이다.
    * AMD나 System과 사용하려면, outFile이 지정되어야 한다.
    * ES6나 ES5를 사용하려면, target이 es5 이하여야 한다.
  * moduleResolution
    * ts 소스에서 모듈을 사용하는 방식을 지정해야 한다.
    * Classic 아니면 Node이다.
    * CommonJS일때만 Node라고 생가하면 된다.
  * paths와 baseUrl
    * 상대경로 방식이 아닌 baseUrl로 꼭지점과 paths 안의 키/밸류로 모듈을 가져가는 방식이다
  * rootDirs 
    * 배열 안에서 상대 경로를 찾는 방식

