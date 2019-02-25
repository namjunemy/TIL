# ES6 for Vue.js

> Intro
>
> * ES6의 여러가지 문법 중 Vue.js 코딩을 간편하게 해주는 문법 학습
> * const & let, Arrow Function, Enhanced Object Literals, Modules 학습

## ES6 란?

* ECMAScript 2015와 동일한 용어
* 2015년은 ES5(2009년)이래로 진행한 첫 메이저 업데이트가 승인된 해
* 최신 Front-End Framework인 React, Angular, Vue에서 권고하는 언어 형식
* ES5에 비해 문법이 간결해져서 익숙해지면 코딩을 훨씬 편하게 할 수 있음

## Babel

* 구 버전 브라우저 중에는 ES6의 기능을 지원하지 않는 브라우저가 있으므로 transpiling이 필요
* ES6의 문법을 각 브라우저의 호환 가능한 ES5로 변환하는 컴파일러

```javascript
module: {
  loaders: [{
    test: /\.js$./,
    loader: 'babel-loader',
    query: {
      presets: ['es2015']
    }
  }]
},
```

## const & let

* 새로운 변수 선언 방식

  * 블록단위 `{ }`로 변수의 범위가 제한되었음
  * `const` : 한번 선언한 값에 대해서 변경할 수 없음(상수 개념)
  * `let` : 한번 선언한 값에 대해서 다시 선언할 수 없음

* 위 특징들에 대해 자세히 알아보기 전에 **ES5 특징 2가지를 리뷰**한다.
  * **ES5 특징 - 변수의 Scope**

    * 기존의 자바스크립트(ES5)는 `{ }`에 상관없이 스코프가 설정됨

      * for문 안의 i를 밖에서 참조할 수 있다.(!!!!!!)

      ```javascript
      var sum = 0;
      for (var i = 1; i <= 5; i++) {
        sum = sum + 1;
      }
      console.log(sum); // 15
      console.log(i);   // 6
      ```

  * **ES5 특징 - Hoisting**

    * Hoisting이란 선언한 함수와 변수를 해석기가 가장 상단에 있는 것처럼 인식한다.

    * js 해석기는 코드의 라인 순서와 관계 없이 **함수 선언식**과 **변수**를 위한 메모리 공간을 먼저 확보한다.

      * 함수 선언식

        ```javascript
        function sum() {
          return 10 + 20;
        }
        ```

      * 함수 표현식

        ```javascript
        var sum = function() {
          return 10 + 20;
        }
        ```

    * 따라서, `function a()` 와 `var`는 코드의 최상단으로 끌어 올려진 것(hoisted)처럼 보인다.

      ```javascript
      function willBeOverriden() {
        return 10;
      }
      willBeOverriden();	// 5
      function willBeOverriden() {
        return 5;
      }
      ```

    * 아래와 같은 코드를 실행할 때 자바스크립트 해석기가 어떻게 코드 순서를 재조정할까?

      * **호이스팅**에 의해서 변수와 함수선언문이 위로 끌어올려지고,
      * 두번째로 **연산**이 일어난다. sum + i
      * 세번째로 **할당**이 일어난다. sum = sum + i
      * 신기하겠지만(js로 시작을 하지 않아서...) 재조정 결과를 보자

      ```javascript
      var sum = 5;
      sum = sum + i;
      function sumAllNumbers() {
        // ...
      }
      
      var i = 10;
      ```

      ```javascript
      // #1 - 함수 선언식과 변수 선언을 hoisting
      var sum;
      function sumAllNumbers() {
        //...
      }
      var i;
      
      // #2 - 변수 대입 및 할당
      sum = 5;
      sum = sum + i;
      i = 10;
      ```

* **ES6 - `{ }` 단위로 변수의 범위가 제한됨**

  ```javascript
  let sum = 0;
  for (let i = 1; i <= 5; i++) {
    sum = sum + i;
  }
  console.log(sum); // 10
  console.log(i);   // Uncaught ReferenceError: i is not defined
  ```

* **ES6 - `const`로 지정한 값 변경 불가능**

  ```javascript
  const a = 10;
  a = 20; // Uncaughr TypeError: Assignment to constant variable
  ```

  * 하지만, 객체나 배열의 내부는 변경할 수 있다.

    ```javascript
    const a = {};
    a.num = 10;
    console.log(a); // {num: 10}
    
    const a = [];
    a.push(20);
    console.log(a); // [20]
    ```

* **ES6 - `let` 선언한 값에 대해 다시 선언 불가능**

  ```javascript
  let a = 10;
  let a = 20; // Uncaught SyntaxError: Identifier 'a' has already been declared
  ```

* **ES6 - const, let**

  ```javascript
  function f() {
    {
      let x;
      {
        // 새로운 블록안에 새로운 x의 스코프가 생김
        const x = "sneaky";
        x = "foo"; // 위에 이미 const로 x를 선언했으므로 다시 값을 대입하면 에러 발생
      }
      // 이전 블록 범위로 돌아왔기 때문에 `let x`에 해당하는 메모리에 값을 대입
      x = "bar";
      let x = "inner"; // Uncaught SyntaxError: Identifier 'x' has already been declared
    }
  }
  ```


## Arrow Function

* 함수를 정의할 때 `function` 이라는 키워드를 사용하지 않고 `=>`로 대체
* 흔히 사용하는 콜백 함수의 문법을 간결화

```javascript
// ES5 함수 정의 방식
var sum = function(a, b) {
  return a + b;
};

// ES6 함수 정의 방식
var sum = (a, b) => {
  return a + b;
}

sum(10, 20);
```

* 화살표 함수 사용 예시

```javascript
// ES5
var arr = ["a", "b", "c"];
arr.forEach(function(value) {
  console.log(value); // a, b, c
});

// ES6
var arr = ["a", "b", "c"];
arr.forEach(value => console.log(value));  // a, b, c
```

* Babel 온라인 에디터로 실습하기 ES6 -> ES5로 변경되는 코드
  * https://babeljs.io/repl/

