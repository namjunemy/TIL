# Vue Instance

> 누구나 다루기 쉬운 Vue.js 프론트 개발 - Captain Pangyo [링크](https://www.inflearn.com/course/vue-pwa-vue-js-%EA%B8%B0%EB%B3%B8/)

## 인스턴스 소개

Vue.js를 이용하여 UI 화면을 개발하기 위해서는 아래 절차를 따른다.

* Vue.js 라이브러리를 로딩했을 때 존재하는 Vue 생성자로 인스턴스를 생성해야 한다.

```js
var vm = new Vue({
	//...
})
```

* 풀어서 얘기하면 Vue 라이브러리 로딩 후 접근 가능한 **Vue**라는 기존 객체에 화면에서 사용할 옵션(데이터, 속성, 메서드 등)을 포함하여 화면의 단위를 생성한다라고 보면 된다.
* 결론적으로 인스턴스는 뷰를 이용해서 개발하는데 가장 중요한 요소이며, 후반부에서 배우게 될 컴포넌트와도 밀접히 연관이 있는 기반 개념이다.

## 인스턴스 생성

* Vue Instance 생성자

  * Vue 생성자로 인스턴스를 만드는 방법은 아래와 같다.

```js
// vm은 ViewModel을 뜻한다.(관행적인 코딩 컨벤션)
var vm = new Vue({
  // options
})
```

* Vue 객체를 생성할 때 아래와 같이 data, template, el, methods, life cycle callback 등의 options를 포함할 수 있다.
  * template : 화면에 나타나는 요소들(div, p, 버튼 등)과 추가적으로 데이터 바인딩에 관한 요소들이 들어간다.
  * el : 화면이 그려지는 시작점이다. element의 약자.  el에 지정한 특정 태그 부터 화면이 그려진다고 보면 된다.
  * methods : 클릭하거나, http 요청 등에 대한 구현 메소드 등을 안에 선언할 수 있다.
  * created : 라이프사이클과 관련된 옵션으로 뒤에서 추가로 설명

```js
var vm = new Vue({
  template: ...,
  el: ...,
  methods: {
  
	},
  created: {

  }
  // ...
})
```

* 각 options 로 미리 정의한 vue 객체를 확장하여 재사용이 가능하다. 하지만 아래 방법 보다는 template에서 custom element로 작성하는 것이 더 좋다.
  * 뒤에서 컴포넌트 학습하면서 이것에 대해 배운다.

```js
var MyComponent = Vue.extend({
  // template, el, method와 같은 options 정의
  template: `<p>Hello {{ message }}</p>`,
  data: {
    message: 'Vue.js !'
  }
  // ...
})

// 위에서 정의한 내용 (template, data, ...)을 기본으로 하는 컴포넌트 생성
var myComponentInstance = new MyComponent()
```

## Vue Instance Lifecycle

* Instance Lifecycle 초기화
* 뷰 객체가 생성될 때 아래의 초기화 작업을 수행한다.
  * 데이터 관찰
  * 템플릿 컴파일
  * DOM에 객체 연결
  * 데이터 변경시 DOM 업데이트
* 위의 초기화 작업 외에도 개발자가 의도하는 커스텀 로직을 아래와 같이 추가할 수 있다.
  * created라는 옵션은 뷰 인스턴스가 생성됐을 때, 원하는 로직을 추가해서 구현할 수 있는 부분이다.

```js
var vm = new Vue({
  data: {
    a: 1
  },
  created: function () {
    console.log('a is: ' + this.a)
  }
})
```

* 이 외에도 Lifecycle 단계에 따라 아래 메서드를 사용할 수 있다.
  * mounted
  * updated
  * destroyed
* 위와 같이 초기화 메서드로 커스텀 로직을 수행하기 때문에 Vue에서는 따로 Controller를 갖고 있지 않다.
* Vue.js 공식 홈페이지의 Lifecycle Diagram - [링크](https://vuejs.org/v2/guide/instance.html#Lifecycle-Diagram)

![](https://vuejs.org/images/lifecycle.png)

## Vue instance lifecycle hook custom logic

**실습**

* **주의할 점**은 updated 속성은 mounted 단계에서 데이터가 변경 되어야만 동작한다는 것이다. created 단계에서 데이터가 변경되면 updated는 동작하지 않게 된다.

```vue
<html>
<head>
  <title>Vue Sample</title>
</head>

<body>
  <div id="app">
    {{ message }}
  </div>

  <script src="https://unpkg.com/vue@2.3.3"></script>
  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        message: 'Hello Vue.js!!'
      },
      beforeCreate: function() {
        console.log("beforeCreate");
      },
      created: function() {
        console.log("created");
      },
      mounted: function() {
        console.log("mounted");
        this.message = '123';
      },
      updated: function() {
        console.log("updated");
      }
    })
  </script>
</body>
</html>
```

