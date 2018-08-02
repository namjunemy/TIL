# Data Binding

* DOM 기반 HTML Template에 Vue 데이터를 바인딩 하는 방법은 아래와 같이 크게 3가지가 있다.
  * Interpolation(값 대입)
  * Binding Expressions(값 연결)
  * Directives(디렉티브 사용)

## Interpolation

값 대입

* Vue의 가장 기본적인 데이터 바인딩 체계는 Mustache `{{ }}`를 따른다.

```html
<span>Message: {{ msg }}</span>
<span>This will never change: {{* msg }}</span> //* 의 의미는 로딩 후 데이터가 변하지 않게 하겠다는 것이다.
<div id="item-{{ id }}"></div>
```

## Binding Expressions

값 연결

* `{{ }}` 를 이용한 데이터 바인딩을 할 때 자바스크립트 표현식을 사용할 수 있다.

```html
<div>{{ number + 1 }}</div> <!-- O -->
<div>{{ message.split('').reverse().join('') }}</div> <!-- O -->

<div>{{ id (ok) { return message } }}</div> <!-- X -->
```

* Vue에 내장된 Filter를 `{{ }}` 안에 사용할 수 있다. 여러개 필터 체인 가능

```html
{{ message | capitalize }}
{{ message | capitalize | upcapitalize }}
```

## Directives

* Vue에서 제공하는 특별한 Arrtibutes 이며 `v-` 의 prefix(접두사)를 갖는다.
* 자바스크립트 표현식, filter 모두 적용된다.

```html
<!-- login의 결과에 따라 p가 존재 또는 미존재 -->
<p v-if="login">Hello!</p>

<!-- click = {{doSomething}} 과 같은 역할 -->
<a v-on:click="doSomething">
```

## Class Binding

* 순수 웹기술로만 개발할 경우에는 클래스에 있는 값들을 자바스크립트로 전달하기가 까다롭다.
* 뷰에서는 mustache 태그를 이용해서 클래스 값을 반응형으로 뷰 인스턴스와 맵핑할 수 있다.
* CSS 스타일링을 위해서 class를 아래 2가지 방법으로 추가가 가능하다.
  * `class="{{ className }}"`
  * `v-bind:class`
* 주의할 점은 위의 두 방법을 함께 사용하지 않고 한 가지만 적용해야 에러를 미연에 방지할 수 있다.
* 아래와 같이 `class` 속성과 `v-bind:class` 속성을 동시에 사용해도 된다.

```html
<div class="static" v-bind:class="{ 'class-a': isA, 'class-b': isB}"></div>
<script>
  data: {
    isA: true,
    isB: flase
  }
</script>
```

* 위 결과 값은

```html
<div class="static class-a"></div>
```

* 아래와 같이 Array 구문도 사용할 수 있다.

```html
<div v-bind:class="[classA, classB]">
  <script>
    data: {
      classA: 'class-a',
      classB: 'class-b'
    }
  </script>
</div>
```

## Template, Data Binding 퀴즈

* index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue Templates Sample</title>
  </head>
  <body>
    <div id="app">
      <header>
        <!-- #1 -->
        <h2>{{ message }}</h3>
        <h3>{{ secondMessage }}</h2>
        <h4>{{ thirdMessage }}</h4>

      </header>
      <section>
        <!-- #2 -->
        <p v-bind:id="uid"></p>

        <!-- #3 -->
        <button v-on:click="clickBtn">console</button>
        <!-- <button @click="clickBtn">alert</button> -->
        <button v-on:click="clickAlert">alert</button>
        <!-- #4 -->
        <ul v-if="flag">
          <li>1</li>
          <li>2</li>
          <li>3</li>
        </ul>
      </section>
    </div>

    <script src="js/vendor/vue.js"></script>
    <script src="js/app.js"></script>
  </body>
</html>
```

* app.js

```js
var app = new Vue({
  el: '#app',
  data: {
    message : 'Hello Vue.js',
    // 할일 #1
    // 새로운 데이터 속성을 1개 추가하고, data bindings 를 이용하여 화면에 표시해보세요.
    secondMessage : 'Bye Vue.js',
    thirdMessage : '3rd Vue.js',
    uid: '26',
    // 할일 #2
    // uid 를 변경하고 해당 uid 의 변경 여부를 크롬 개발자 도구의 "화면 요소 검사" 기능으로 p 태그의 id 값을 확인해보세요.

    flag: false
    // 할일 #4
    // 위 flag 값을 false 로 변경하였을 때 화면에 어떤 영향을 주는지 확인해보세요.
  },
  methods: {
    // ES6
    clickBtn() {
      console.log("hi");
    },
    // ES5
    // clickBtn: function() {
    //   console.log("hi");
    // }

    // 할일 #3
    // eventMethod 를 하나 추가하고 template 에서 해당 이벤트를 실행할 수 있는 button 을 하나 추가하세요.
    clickAlert() {
      alert("Vue Vue Vue");
    }
  }
});
```

