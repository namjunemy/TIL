# Vue Component 통신

> Inflearn - Captain Pangyo

## 상-하위 컴포넌트 간 데이터 전달 방법

* 구조상 상-하 관계에 있는 컴포넌트의 통신은
  * 부모 -> 자식 : props down
  * 자식 -> 부모 : events up

## Props

* 모든 컴포넌트는 각 컴포넌트 자체의 스코프를 갖는다.
  * ex) 하위 컴포넌트가 상위 컴포넌트의 값을 바로 참조할 수 없는 형식
  * 위의 예가 props를 사용하는 이유이다. 
* 상위에서 하위로 값을 전달하려면 **props 속성**을 사용한다.

#### props 예제

* 주의할 점
  * js에서 props 변수 명명을 카멜 기법으로 하면 html에서 접근은 케밥 기법(`-`)으로 가야한다.
* 스페셜 프로퍼티인 뷰의 v-bind 속성을 이용하여 컴포넌트에서 props를 받아서 사용할 수 있다.
* 정리하면, app이라는 id를 가진 div 태그의 child-component를 뷰가 그릴 때, v-bind 키워드를 이용하여 전역 컴포넌트의 props로 선언된 passedData를 참조하고, 그 값은 인스턴스(부모 컴포넌트)의 data에 선언된 message가 전달 된다. 마지막으로 template에는 message가 담긴 p태그가 들어간다.

```js
Vue.component('child-component', {
  props: ['passedData'],
  template: '<p>{{ passedData }}</p>'
});

var app = new Vue({
  el: '#app',
  data: {
  message: 'Hello Vue! passed from Parent Component'
  }
});
```

```html
<div id="app">
  <child-component v-bind:passed-data="message"></child-component>
</div>
```

## Props 실습 퀴즈

* app.js

```js
// 할일 #1
// sibling-component 를 이름으로 갖는 새로운 컴포넌트를 아래에 등록해보세요.
// options : template, props
Vue.component('sibling-component', {
  props: ['anotherData'],
  template: '<p>{{anotherData}}</p>'
})

Vue.component('child-component', {
  props: ['passedData'],
  template: '<p>{{passedData}}</p>'
});

var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue! passed from Parent Component',
    anotherMessage: 'This is an anotherMessage!'
    // 할일 #2
    // data 속성을 한개 더 지정하고 (예, anotherMessage) 임의의 문자열을 값으로 대입해보세요.
    // 새로 지정한 data 속성을 위 sibling-component 에 props 로 전달합니다.
  }
});
```

* index.html
  * v-bind:[props 명]=[상위컴포넌트의 데이터]

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue Props Sample</title>
  </head>
  <body>
    <div id="app">
      <!-- passedData (props) equals to be passed-data  -->
      <!-- read the bind property from right to left (from bottom to top in app.js)-->
      <child-component v-bind:passed-data="message"></child-component>

      <!-- #할일 3 -->
      <!-- sibling-component 등록 -->
      <sibling-component v-bind:another-data="anotherMessage"></sibling-component>
    </div>

    <script src="js/vendor/vue.js"></script>
    <script src="js/app.js"></script>
  </body>
</html>
```

## Non Parent-Child 컴포넌트 간 통신

* 같은 레벨의 컴포넌트 간 통신
  * 동일한 상위 컴포넌트를 가진 2개의 하위 컴포넌트 간의 통신은
  * Child -> Parent -> 다시 2개의 Child
  * 컴포넌트 간의 직접적인 통신은 불가능하도록 되어 있는게 Vue의 기본 구조이다.

## Event Bus - 컴포넌트 간 통신

Non Parent-Child 컴포넌트 간의 통신을 위해 Event Bus를 활용할 수 있다. 장점 이것을 통해 화면에 있는 어떤 컴포넌트들 간에도 통신을 할 수 있다.

단점은 컴포넌트들 간의 관계가 명확해지지 않는다. Parent-Child 관계는 상위, 하위로 명확하지만 예를 들어, child끼리의 관계에서는 이벤트 정리가 어려워진다. 이 내용에 대해서는 vuex를 통해서 학습한다. vuex는 컴포넌트 간의 이벤트나 데이터 관리를 용이하게 하기 위한 라이브러리이다. Event Bus가 vuex의 필요성을 높이고 있다.

* Event Bus를 위해 새로운 Vue를 생성하여 아래와 같이 Vue Root Instance가 위치한 파일에 등록한다.
  * 화면단을 구성하는 뷰 인스턴스와 별개로 비어있는 뷰 인스턴스를 하나 생성해서 컴포넌트 간 통신의 중개자 역할로 사용한다.

```vue
//Vue Root Instance 전에 꼭 등록 순서가 중요
export const eventBus = new Vue();
new Vue({
	// ...
})
```

### 이벤트 발생

* 이벤트를 발생시킬 컴포넌트에 `eventBus` import 후 `$emit` 으로 이벤트 발생

```js
import { eventBus } from '../../main';
eventBus.$emit('refresh', 10);
```

### 이벤트 수신

* 해당 이벤트를 받을 컴포넌트에도 동일하게 import 후 콜백으로 이벤트 수신

```js
import { eventBus } from '../../main';

//등록 위치는 해당 컴포넌트의 created 메서드에 등록
created() {
  eventBus.$on('refresh', function (data) {
    console.log(data); //10
  });
}
```

## Component - Props - For 퀴즈

* v-for를 이용할 때, 리스트에서 삭제나 변경이 이루어지면 중간에 요소가 빠지게 되고 나머지 리스트가 순서를 재조정하게 되는데, 이 경우 브라우저의 부하를 줄이기 위해서 :key를 명시적으로 선언하게 되면 DOM을 옮기지 않고(DOM을 옮기지 않는다는 말은 요소의 움직임을 최소화 해서 브라우저의 부하를 줄이겠다는 의미이다.) 필요한 것만 업데이트 할 수 있다.
  * 결론은 v-for를 이용할 때, :key값을 명시적으로 선언하면 더 빠른 랜더링을 할 수 있다.
* app.js

```js
// 할일 #2
// 아래에 todo-list 컴포넌트를 구현
Vue.component('todo-list', {
  props: ['language'],
  template: '<p>{{ language.value }}</p>'
});

Vue.component('todo-item', {
  props: ['todo'],
  template: '<p>{{ todo.text }}</p>'
});

var app = new Vue({
  el: '#app',
  data: {
    Todos: [
      { id: 0, text: 'Learn Vue.js' },
      { id: 1, text: 'Learn Components' },
      { id: 2, text: 'Learn Props' },
      { id: 3, text: 'Learn For Loop' }
    ],
    // 할일 #1
    // 배열 안에 python, c++, java, objective-c 를 값으로 갖는
    // Languages 프로퍼티를 추가하여 위에 제작한 todo-list 컴포넌트에 전달하고,
    // 배열 값을 모두 for loop 로 출력하세요.
    Languages: [
      { id: 0, value: 'python'},
      { id: 1, value: 'c++'},
      { id: 2, value: 'java'},
      { id: 3, value: 'objective-c'}
    ]
  }
});
```

* index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue Components-Props-For Sample</title>
  </head>
  <body>
    <!-- Parent Component -->
    <div id="app">
      <!-- Child Component -->
      <todo-item v-bind:todo="todo" v-for="todo in Todos" :key="todo.id"></todo-item>

      <!-- 할일 #3 -->
      <!-- todo-list 컴포넌트 등록 -->
      <todo-list v-bind:language="language" v-for="language in Languages" :key="language.id"></todo-list>
    </div>
    <script src="js/vendor/vue.js"></script>
    <script src="js/app.js"></script>
  </body>
</html>
```

