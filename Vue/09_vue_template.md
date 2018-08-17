# Vue Templates

> 누구나 다루기 쉬운 Vue.js 프론트 개발 - Captain Pangyo [링크](https://www.inflearn.com/course/vue-pwa-vue-js-%EA%B8%B0%EB%B3%B8/)

Vue로 그리는 화면의 요소들, 함수, 데이터 속성은 모두 Templates 안에 포함된다.

## Intro

* Vue는 DOM의 요소와 Vue 인스턴스를 매핑할 수 있는 HTML Template을 사용
  * html 표준 태그들과 함께 Vue의 스페셜 프로퍼티들(v-bind, v-for 등)을 함께 사용했을 때, 마지막에 브라우저가 그려내는 것들에는 뷰의 반응적인 속성들까지 결합된 화면이 그려져 나온다. 예를 들면, 라이프 사이클에서 얘기했던 것처럼, 뷰에 맵핑된 데이터를 가지고 있는 템플릿 태그의 데이터가 변환이 되면 자동적으로 태그가 DOM을 업데이트 한다. 화면의 요소를 자동으로 갱신한다고 보면 된다.
* Vue는 Template으로 렌더링 할 때 Virtual DOM을 사용하여 DOM 조작을 최소화 하고 렌더링을 꼭 다시 해야만 하는 요소를 계산하여 성능 부하를 최소화 한다.(React와 동일한 장점)
* 원하면 **render function**을 직접 구현하여 사용할 수 있다.
  * 템플릿 안에 들어가는 html요소들, 뷰와 관계되는 스페셜 프로퍼티들이 마지막에 render function으로 대체가 되서 화면이 그려지게 되는데, 이것도 결국에 React의 기능이다. 뷰도 DOM 조작을 직접적으로 관여해서 성능을 최적화 할 수 있다는 render function의 장점을 가저간다.

> render function 학습 링크
>
> https://vuejs.org/v2/guide/render-function.html

## Attributes

* HTML Attributes를 Vue의 변수와 연결할 때는 `v-bind`를 이용
* 코드상에만 존재하는 뷰의 스페셜 프로퍼티로 실제 페이지에는 보여지지 않는 임시 값이다.

```html
<div v-bind:id="dynamicId"><div>
```

## Javascript Expressions

* 뷰에서는 mustache `{{ }}` 안에 다음과 같이 javascript 표현식도 가능하다.
  * 분기문이나 복잡한 표현식은 허용하지 않는다.

```html
<div>{{ number + 1 }}</div> <!-- O -->
<div>{{ message.split('').reverse().join('') }}</div> <!-- O -->

<div>{{ if (ok) { return message } }}</div> <!-- X -->
```

 ## Directives

`v-` 접두사를 붙인 attributes로, javascript 표현식으로 값을 나타내는게 일반적이다. `:` 을 붙여 인자를 받아 취급할 수 있다.

* v-on:click은 태그가 클릭됐을 때, doSomething이라는 특정 메서드를 호출하게 된다.

```js
<!-- true일 때는 보이고, false일 때는 보이지 않는다. -->
<p v-if="seen">Now you see me</p>

<!-- : 뒤에 선언한 href 인자를 받아 url 값이랑 매핑 -->
<a v-bind:href="url"></>

<!-- click 이라는 이벤트를 받아 Vue에 넘겨준다. -->
<a v-on:click="doSomething">
```

## Filters

화면에 표시되는 텍스트의 형식을 편하게 바꿀 수 있도록 고안된 기능이며, `|` 을 이용하여 여러개의 필터를 적용할 수 있다.

```html
<!-- message에 표시될 문자에 capitalize 필터를 적용하여 첫 글자를 대문자로 변경한다. -->
{{ message | capitalize }}
```

```js
new Vue({
  // ...
  filters: {
    capitalize: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
})
```

