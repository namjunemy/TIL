# TypeScript Basic Types

>  '인프런 - 타입스크립트 코리아: 기초 세미나'를 학습한 내용입니다.



* 타입스크립트에서 프로그램 작성을 위해 기본 제공하는 데이터 타입
* **사용자가 만든 타입은 결국 이 기본 자료형들로 쪼개진다.**
* JavaScript 기본 자료형을 포함한다.(Superset)
  * ECMAScript 표준에 따른 기본 자료형은 6가지
    * Boolean
    * Number
    * String
    * Null
    * Undefined
    * Symbol (ECMAScript 6에 추가)
    * Array : object형
* 프로그래밍을 도울 몇가지 타입이 더 제공된다.
  * Any
  * Void
  * Never
    * 위의 세가지는 함수의 리턴타입으로 많이 사용된다.
  * Enum
  * Tuple : object 형



### Primitive Type

* 오브젝트와 레퍼런스 형태가 아닌 실제 값을 저장하는 자료형
* primitive 형의 내장 함수를 사용 가능한것은 자바스크립트 처리 방식 덕분이다.



### Literal

* 값자체가 변하지 않는 값을 의미한다.
* 상수와 다른 것은 상수는 가리키는 포인터가 고정이라는 것이고, 리터럴은 그 자체가 값이자 그릇이다.



### Boolean / boolean

* 가장 기본적인 데이터 타입
* 단순한 true / false 값이다.
* 자바스크립트 / 타입스크립트에서 'boolean'이라고 부른다.



### Number / number

* new Number()로 생성한 타입에 number타입을 넣을 수 없고,
* 소문자 number를 쓰는 것을 권장한다.


* 자바스크립트와 같이, 타입스크립트의 모든 숫자는 부동 소수점 값이다.



### String / string

* 텍스트 형식을 참조하기 위해 string 형식을 사용한다.
* 자바스크립트와 마찬가지로 타입스크립트는 문자열 데이터를 둘러싸지 위해 " 나 ' 를 사용한다. 


* **Template String**을 사용한다.
  * 행에 걸쳐 있거나, 표현식을 넣을 수 있는 문자열
  * 이 문자열은 backpack (= backquote) 기호에 둘러쌓여 있따.
  * 포함된 표현식은 \`${expr}\`와 같은 형태로 사용한다.



### undefined & null

* 타입스크립트에서 undefined 와 null은 실제로 각각 undefined와 null 이라는 고유한 타입을 가진다.
* void 와 마찬가지로, undefined와 null은 그 자체로는 쓸모가 없다. 리턴타입으로 주로 사용한다.
* 둘다 소문자만 존재한다.
* **undefined 와 null 은 다른 모든 타입의 sub type이다**
  * 기본 설정이 그렇다
  * number에 null 또는 undefined를 할당할 수 있다는 의미니다.
  * 하지만, 컴파일 옵션에서 \`—strictNullChecks\`를 사용하면 null과 undefined는 void나 자기 자신들에게만 할당할 수 있다.
    * 이 경우에 null과 undefined를 할당할 수 있게 하려면 union type을 이용해야 한다.
    * union type은 두개의 타입을 합치는 것이다. 추후에 학습한다.
* null in JavaScript
  * null이라는 값으로 할당된 것을 null 이라고 한다.
  * 무언가가 있는데, 사용할 준비가 덜 된 상태.
  * null이라는 타입은 null이라는 값만 가질 수 있다.
  * 런타임에서 typeof 연산자를 이용해서 알아내면, object이다.
* undefined in JavaScript
  * 값을 할당하지 않은 변수는 undefined 라는 값을 가진다.
  * 무언가가 아예 준비가 안된 상태이다.
  * object의 property가 없을 때도 undefined이다.
  * 런타임에서 typeof 연산자를 이용해서 알아내면, undefined이다.



### void

* 타입이 없는 상태
* any 와 반대의 이미를 가진다.
* void이다.
* 주로 함수의 리턴이 없을 때 사용한다.



### Any

* 어떤 타입을 넣어도 되기 때문에, 헬퍼를 얻어 낼 수 없는게 단점이다.
* 이걸 최대한 쓰지 않는게 핵심이다.
* 컴파일 타임에 타입 체크가 정상적으로 이뤄지지 않기 때문이다.
* 그래서 컴파일 옵션중에 any를 쓰면 오류를 뱉도록 하는 옵션도 있다.
  * noImplicitAny



### Never

* 리턴에 주로 사용된다.



### Array

* 원래 자바스크립트에서 Array는 객체이다.
* 사용방법
  * Array<타입>
  * 타입[]



### Tuple

* 배열인데 타입이 한가지가 아닌 경우
* 마찬가지로 객체이다.
* 꺼내 사용할때 주의가 필요하다.



### Enum

* C의 Enum과 같다
* Enum의 원소의 결과값은 String형 이다.



### Symbol

* ECMAScript 2015의 Symbol 이다.
* 프리미티브 타입의 값을 담아서 사용한다.
* 고유하고 수정불가능한 값으로 만들어 준다.
* 그래서 주로 접근을 제어하는데 쓰는 경우가 많았다.