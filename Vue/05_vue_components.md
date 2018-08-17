# Vue Components

> 누구나 다루기 쉬운 Vue.js 프론트 개발 - Captain Pangyo [링크](https://www.inflearn.com/course/vue-pwa-vue-js-%EA%B8%B0%EB%B3%B8/)

컴포넌트는 화면에 비춰지는 뷰의 단위를 쪼개어 재활용이 가능한 형태로 관리하는 것이 주된 목적이다.

![](https://github.com/namjunemy/TIL/blob/master/Vue/img/03.PNG?raw=true)

## 컴포넌트 등록

* 뷰 컴포넌트를 등록한 후, 뷰 인스턴스를 생성하면 등록된 컴포넌트에 의해서 my-component가 컴포넌트의 template 내용으로 치환되면서 화면에 뿌려진다.

```vue
<html>
<head>
  <title>Vue Sample</title>
</head>

<body>
  <div id="app">
    <button>Parent Component</button>
    <my-component></my-component>
  </div>

  <script src="https://unpkg.com/vue@2.3.3"></script>
  <script>
    Vue.component('my-component', {
      template: '<div>A custom component!</div>'
    });

    new Vue({
      el: '#app'
    })
  </script>
</body>
</html>
```

## 전역 & 지역 컴포넌트 등록

* 컴포넌트를 뷰 인스턴스에 등록해서 사용할 때 다음과 같이 global 하게 등록할 수 있다.

```js
Vue.component('my-component', {
	//...
})
```

* local하게 등록하는 방법은 다음과 같다
  * 인스턴스를 생성할 때, 해당 인스턴스에서만 활용할 수 있는 로컬 컴포넌트를 components에 등록한다.

```js
var cmp = {
  data: function () {
    return {
      //...
    };
  }
  template: '<hr>',
  methods: {}
}

// 아래 Vue 인스턴스에서만 활용할 수 있는 로컬(지역) 컴포넌트 등록
new Vue({
  components: {
    'my-cmp' : cmp
  }
})
```

* 전역, 지역 컴포넌트 등록 테스트

```vue
<html>
<head>
  <title>Vue Sample</title>
</head>

<body>
  <div id="app">
    <button>Parent Component</button>
    <my-component></my-component>
    <my-local-component></my-local-component>
  </div>

  <script src="https://unpkg.com/vue@2.3.3"></script>
  <script>
    // 전역 컴포넌트
    Vue.component('my-component', {
      template: '<div>A global component!</div>'
    });

    // 지역 컴포넌트
    var cmp = {
      template: '<div>A local component!</div>'
    };

    new Vue({
      el: '#app',
      components: {
        'my-local-component': cmp
      }
    })
  </script>
</body>
</html>

```

* 전역, 지역 컴포넌트의 차이점
  * 아래의 코드에서 app2의 my-local-component는 치환되지 않는다. 해당하는 전역 컴포넌트가 없기 때문이다. 
  * 따라서, app처럼 해당 인스턴스에서 사용 가능하도록 로컬 컴포넌트로 등록해야 한다.

```vue
<html>
<head>
  <title>Vue Sample</title>
</head>

<body>
  <div id="app">
    <button>Parent Component</button>
    <my-component></my-component>
    <my-local-component></my-local-component>
  </div>

  <div id="app2">
    <my-component></my-component>
    <my-local-component></my-local-component>
  </div>

  <script src="https://unpkg.com/vue@2.3.3"></script>
  <script>
    // 전역 컴포넌트
    Vue.component('my-component', {
      template: '<div>A global component!</div>'
    });

    // 지역 컴포넌트
    var cmp = {
      template: '<div>A local component!</div>'
    };

    new Vue({
      el: '#app',
      components: {
        'my-local-component': cmp
      }
    })

    new Vue({
      el: '#app2'
    })
  </script>
</body>
</html>
```

## 지역 & 전역 컴포넌트 등록 퀴즈

* index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue Components Sample</title>
  </head>
  <body>
    <!-- Parent Component -->
    <div id="app">
      <header>
        <h3>{{ message }}</h3>
      </header>
      <todo-item></todo-item>
      <!-- 실습 #3 - 컴포넌트 등록을 위한 `todo-footer` 태그 추가 -->
      <todo-footer></todo-footer>
    </div>

    <script src="js/vendor/vue.js"></script>
    <script src="js/app.js"></script>
  </body>
</html>
```

* app.js

```js
// Global Component
Vue.component('todo-item', {
  template: '<p>This is a child component</p>'
});

// 실습 #1 - `todo-footer` 컴포넌트 전역 등록
// <p>This is another child global component</p> 를 template 으로 갖는 컴포넌트를 등록해보세요.
Vue.component('todo-footer', {
  template: '<p>This is another child global component</p>'
});

var cmp = {
  template: '<p>This is another child local component</p>'
}

var app = new Vue({
  el: '#app',
  data: {
    message : 'This is a parent component'
  },
  // 실습 #2 - `todo-footer` 컴포넌트 지역 등록
  // <p>This is another child local component</p> 를 template 으로 갖는 컴포넌트를 등록해보세요.
  components: {
    'todo-footer': cmp
  }
});
```

