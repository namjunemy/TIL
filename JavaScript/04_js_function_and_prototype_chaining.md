





# 04. 함수와 프로토타입 체이닝

> 인사이드 자바스크립트

## 4-1 함수 정의

자바스크립트에서 함수를 생성하는 방법은 3가지가 있다.

* 함수 선언문
* 함수 표현식
* Function() 생성자 함수

### 4-1-1 함수 리터럴

* function 키워드
* 함수명
* 매개변수 리스트
* 함수 몸체

```javascript
function add(x, y) {
  return x + y;
}
```

### 4-1-2 함수 선언문 방식으로 함수 생성

함수 선언문 방식은 함수 리터럴 형태와 같다. 주의할 점은 함수 선언문 방식으로 정의된 함수의 경우는 반드시 함수명이 정의되어 있어야 한다는 것이다.

```javascript
function add(x, y) {
  return x + y;
}

console.log(add(4, 5))
```

### 4-1-3 함수 표현식 방식으로 함수 생성하기

함수 리터럴로 하나의 함수를 만들고, 여기서 생성된 함수를 변수에 할당하여 함수를 생성하는 것을 함수 표현식이라고 말한다. 함수 리터럴로 생성한 함수는 함수명이 없으므로 익명 함수이다. 유일한 차이점은 함수 표현식 방법에서는 함수 이름이 선택사항이며, 보통 사용하지 않는다는 것이다.

* add와 plus 함수는 두 개의 인자를 더하는 동일한 익명 함수를 참조한다.

```javascript
var add = function(x, y) {
  return x + y;
};

var plus = add;

console.log(add(3, 4));
console.log(plus(5, 6));
```

* 아래의 경우처럼 기명 함수 표현식으로 사용할 수도 있다. 하지만 주의사항이 있다.
  * 함수 표현식에서 사용된 함수 이름이 외부 코드에서 접근 불가능하다.

```javascript
var add = function sum(x, y) {
  return 
};

console.log(add(3, 4));  // (출력값) 7
console.log(sum(3, 4));  // (출력값) Uncautch ReferenceError: sum is not defined 에러 발생
```

* 하지만, 아래의 선언문으로 정의한 add() 함수는 자바스크립트 엔진에 의해 다음과 같은 함수 표현식으로 변경되기 때문에 외부에서 호출이 가능하게 된다.

```javascript
function add(x, y) {
  return x + y;
}
```

```javascript
var add = function add(x, y) {
  return x + y;
}
```

* 기명 함수 표현식을 사용하면 함수 코드 내부에서 함수 이름으로 함수의 재귀적인 호출 처리가 가능하다.

```javascript
var factorialVar = function factorial(n) {
  if (n <= 1) {
    return 1;
  }
  return n * factorial(n-1);
}

console.log(factorialVar(3));
console.log(factorial(3));  // (출력값) Uncautch ReferenceError: sum is not defined 에러 발생
```

### 4-1-4 Function() 생성자 함수를 통한 함수 생성하기

자바스크립트의 함수도 Function() 이라는 기본 내장 생성자 함수로부터 생성된 객체라고 볼 수 있다. 위에서 설명한 함수 선언문이나 함수 표현식 방식도 Function() 생성자 함수가 아닌 함수 리터럴 방식으로 함수를 생성하지만, 결국엔 이 또한 내부적으로는 Function() 생성자 함수로 함수가 생성된다고 볼 수 있다.

일반적으로 Function() 생성자 함수는 자주 사용되지 않는다.

```javascript
var add = new Function('x', 'y', 'return x + y');
console.log(add(3, 4));
```

### 4-1-5 함수 호이스팅

지금까지 함수를 생성하는 방법을 살펴봤다. 하지만 이들 사이에는 동작 방식이 약간 차이가 있다. 그중의 하나가 바로 함수 호이스팅이다. 자바스크립트 Guru로 알려진 더글러스 크락포드는 함수 생성에 있어서 함수 표현식만을 사용할 것을 권하고 있다. 그 이유중의 하나가 바로 함수 호이스팅 때문이다.

아래의 코드를 보자.

1번 라인에 아직 add() 함수가 정의되지 않았음에도 add() 함수를 호출하는 것이 가능하다. **이것은 함수가 자신이 위치한 코드에 상관없이 함수 선언문 형태로 정의한 함수의 유효번위는 코드의 맨 처음부터 시작한다는 것을 확인할 수 있다.** 이것을 함수 호이스팅 이라고 부른다.

```javascript
add(2, 3);

function add(x, y) {
  return x + y;
}

add(3, 6);
```

더글러스 크락포드는 이러한 함수 호스팅은 함수를 사용하기 전에 반드시 선언해야 한다는 규칙을 무시하므로 코드의 구조를 엉성하게 만들 수도 있다고 지적하며, **함수 표현식 사용을 권장**하고 있다.

다음과 같이 함수 표현식 형태로 add() 함수를 정의하면 실행결과는 다르게 된다.

add() 함수는 함수표현식 형태로 정의되어 있어 호이스팅이 일어나지 않는다. 따라서, 함수가 생성된 이후에 호출이 가능하다.

```javascript
add(2, 3);

var add = function (x, y) {
  return x + y;
};

add(3, 6);

```

  

## 4-2 함수 객체 : 함수도 객체다

### 4-2-1 자바스크립트레서는 함수도 객체다

자바스크립트에서는 함수도 객체다. 즉, 함수의 기본 기능인 코드 실행뿐만 아니라, 함수 자체가 일반 객체처럼 프로퍼티들을 가질 수 있다는 것이다.

### 4-2-2 함수는 값으로 취급된다

자바스크립트에서 함수는 객체다. 때문에 자바스크립트 함수는 다음과 같은 동작이 가능하다.

* 리터럴에 의해 생성
* 변수나 배열의 요소, 객체의 프로퍼티 등에 할당 가능
* 함수의 인자로 전달 가능
* 함수의 리턴값으로 리턴 가능
* 동적으로 프로퍼티를 생성 및 할당 가능

이와 같은 특징이 있으므로 자바스크립트에서는 함수를 **일급(First Class) 객체**라고 부른다. 여기서 일급 객체라는 말은 컴퓨터 프로그래밍 언어 분야에서 쓰이는 용어로서, 앞에서 나열한 기능이 모두 가능한 객체를 일급 객체라고 부른다. 자바스크립트 함수가 가지는 이러한 일급 객체의 특성으로 함수형 프로그래밍이 가능하다. 이에 대해서는 뒤에서 자세히 다룬다.

자바스크립트 함수의 기능은 C나 자바와 같은 다른 언어 함수의 기능과 거의 비슷하다. 입력한 값을 받아 처리한 다음 그 결과를 반환하는 구조다. 하지만 이러한 기본적인 기능 외에도 자바스크립트에서 함수를 제대로 이해하려면 함수가 일급 객체이며 이는 곧 함수가 일반 객체처럼 값으로 취급된다는 것을 이해해야 한다.

따라서 함수를 변수나 객체, 배열 등에 값으로도 저장할 수 있으며, 다른 함수의 인자로 전달한다거나 함수의 리턴값으로도 사용 가능하다는 것을 알 수 있다.

#### 4-2-2-1 변수나 프로퍼티의 값으로 할당

함수는 숫자나 문자열처럼 변수나 프로퍼티의 값으로 할당될 수 있다.

* 변수에 함수 할당
* 프로퍼티에 함수 할당

```javascript
var foo = 100;
var bar = function () {
  return 100;
};
console.log(bar());

var obj = {};
obj.baz = function () {
  return 200
};
console.log(obj.baz());
```

#### 4-2-2-2 함수 인자로 전달

foo() 함수를 호출할 때, 함수 리터럴 방식으로 생성한 익명 함수를 func 인자로 넘겼다. 따라서 foo() 함수 내부에서는 func 매개변수로 인자에 넘겨진 함수를 호출할 수 있다.

```javascript
var foo = function (func) {
  func();
};

foo(function () {
  console.log('Function can be used as the argument');
});
```

#### 4-2-2-3 리턴 값으로 활용

함수는 다른 함수의 리턴값으로 활용할 수 있다. **이것이 가능한 이유 또한 함수 자체가 값으로 취급되기 때문이다.**

```javascript
var foo = function() {
  return function() {
    console.log('this function is the return value.');
  };
};

var bar = foo();
bar();
```

### 4-2-3 함수 객체의 기본 프로퍼티

계속 강조했듯이 자바스크립트에서 함수 역시 객체다(굉장이 중요한 개념이므로 자꾸 반복한다고 저자가 말한다. 이것은 함수 역시 일반적인 객체의 기능에 추가로 호출됐을 때 정의된 코드를 실행하는 기능을 가지고 있다는 것이다. 또한, 일반 객체와는 다르게 추가로 **함수 객체만의 표준 프로퍼티**가 정의되어 있다.

아래의 예제 코드를 통해서 add() 함수는 arguments, caller, length 등과 같은 다양한 프로퍼티가 기본적으로 생성된 것을 알 수 있다. 이러한 프로퍼티들이 함수를 생성할 때 포함되는 표준 프로퍼티다.

![](https://github.com/namjunemy/TIL/blob/master/JavaScript/img/01_function_basic_property.PNG?raw=true)

* ECMA5 스크립트 명세에서는 모든 함수가 **length**와 **prototype** 프로퍼티를 가져야 한다고 기술하고 있다. 뒷부분에서 자세히 알아본다.
* length나 prototype 이외의 프로퍼티들은 ECMA 표준이 아니다.
* **name 프로퍼티**는 함수의 이름을 나타낸다. 익명 함수라면 name 프로퍼티는 빈 문자열이 된다.
* **caller 프로퍼티**는 자신을 호출한 함수를 나타낸다. 여기서는 함수를 호출하지 않았으므로 null 값이 나왔다.
* **arguments 프로퍼티**는 함수를 호출할 때 전달된 인자값을 나타낸다. 현재 add() 함수가 호출된 상태가 아니므로 null 값이 출력됐다.
  * 뒤에서 알아볼 ECMA 표준에서 정의하고 있는 arguments 객체와 같은 이름이다. 4-4-1 arguments 객체는 함수를 호출할 때 호출된 함수의 내부로 인자값과 함께 전달되며, arguments 프로퍼티와 유사하게 함수를 호출할 때 전달 인자값의 정보를 제공해준다.
* 앞서 설명했지만 add() 함수 역시 자바스크립트 객체이므로 **[[Prototype]] 프로퍼티(크롬 브라우저에서\_\_proto\_\_ 프로퍼티)**를 가지고 있고, 이를 통해 자신의 부모 역할을 하는 프로토타입 객체를 가리킨다.
  * ECMA 표준에서는 add()와 같이 함수 객체의 부모 역할을 하는 프로토타입 객체를 **Function.prototype 객체** 라고 명명하고 있으며, 이것 역시 **함수 객체** 라고 정의하고 있다.

* **Function.prototype 객체의 프로토타입 객체는?**
  * 앞에서 '모든 함수들의 부모 객체는 Function Prototype 객체'라고 했다. 그런데 ECMAScript 명세서에는 Function.prototype은 함수라고 정의하고 있다. 추가적으로 ECMAScript 명세서에는 예외를 설명하고있다. Function.prototype 함수 객체의 부모는 자바스크립트의 모든 객체의 조상격인 Object.prototype 객체라고 설명한다. 때문에 위의 예제 코드에서 add() 함수의 \_\_proto\_\_ 프로퍼티의 \_\_proto\_\_ 프로퍼티는 ㅒObject객체를 가리키고 있는 것이다.
* **Function.prototype 객체는 모든 함수들의 부모 역할**을 하는 프로토타입 객체다. 때문에 모든 함수는 Function Prototype 객체가 있는 프로퍼티나 메서드를 자신의 것처럼 상속받아 그대로 사용할 수 있다.
  * ECMAScript 명세서에는 이러한 Function.prototype 객체가 가져야 하는 프로퍼티들을 다음과 같이 기술하고있다.
    * constructor 프로퍼티
    * toString() 메서드
    * apply(thisArg, argArray) 메서드
    * call(thisArg, [, arg1 [,arg2, ]]) 메서드
    * bind(thisArg, [, arg1 [,arg2, ]]) 메서드5
  * 여기서 apply(), call()는 실제로 자주 사용되는 메서드이다. 4.4.2.4에서 자세히 살펴본다.

#### 4-2-3-1 length 프로퍼티

함수 객체의 length 프로퍼티는 ECMAScript에서 정한 모든 함수가 가져야 하는 표준 프로퍼티로서, 함수가 정상적으로 실행될 때 기대되는 인자의 개수를 나타낸다.

```javascript
function func0() {

}

function func1(x) {
  return x;
}

function func2(x, y) {
  return x + y;
}

function func3(x, y, z) {
  return x + y + z;
}

console.log('func0.length = ' + func0.length);
console.log('func1.length = ' + func1.length);
console.log('func2.length = ' + func2.length);
console.log('func3.length = ' + func3.length);
```

```text
func0.length = 0
func1.length = 1
func2.length = 2
func3.length = 3
```

#### 4-2-3-2 prototype 프로퍼티

모든 함수는 객체로서 prototype 프로퍼티를 가지고 있다. 여기서 주의할 것은 함수 객체의 prototype 프로퍼티는 앞서 설명한 모든 객체의 부모를 나타내는 내부 프로퍼티인 [[Prototype]]과(위의 예제 코드의 결과 중 \_\_proto_\_ ) 혼동하지 말아야 한다는 것이다.

* prototype 프로퍼티와 [[Prototype]] 프로퍼티
  * 두 프로퍼티 모두 프로토타입 객체를 가리킨다는 점에서 공통점이 있지만, 관점에 차이가 있다. 모든 객체에 있는 내부 프로퍼티인 [[Prototype]] 프로퍼티는 객체 입장에서 자신의 부모 역할을 하는 프로토타입 객체를 가리키는 반면에, 함수 객체가 가지는 prototype 프로퍼티는 이 함수가 생성자로 사용될 때 이 함수를 통해 생성된 객체의 부모 역할을 하는 프로토타입 객체를 가리킨다. 4-5-1에서 좀 더 자세히 알아본다.

prototype 프로퍼티는 함수가 생성될 때 만들어지며, constructor 프로퍼티 하나만 있는 객체를 가리킨다. 그리고 prototype 프로퍼티가 가리키는 프로토타입 객체의 유일한 constructor 프로퍼티는 자신과 연결된 함수를 가리킨다.

**즉, 자바스크립트에서는 함수를 생성할 때, 함수 자신과 연결된 프로토타입 객체를 동시에 생성하며, 이 둘은 다음 그림처럼 각각 prototype과 constructor라는 프로퍼티로 서로를 참조하게 된다.** 이 개념은 이후 프로토타입과 프로토타입 체이닝을 이해하는 기본지식인 만큼 잘 알아둬야 한다.

![](https://github.com/namjunemy/TIL/blob/master/JavaScript/img/02_prototype_property.PNG?raw=true)