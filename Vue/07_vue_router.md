# Vue Routers

> Inflearn - Captain Pangyo

## Intro

일반적으로 뷰에서 화면이 전환될 때, 전환하는 행위를 Route라고 표현한다. 뷰에서는 SPA를 제작할 때 유용한 라우팅 라이브러리로 Vue Routers를 제공하고 있다.

> **SPA에 대하여**
>
> 일반적으로 웹 어플리케이션은 해당 웹 페이지를 서버에 요청하고, 서버가 html을 돌려 줬을 때 그것을 랜더링 엔진을 통해 화면에 보여준다. SPA는 해당 화면에 대한 정보를 미리 가지고 있다가, 해당 화면으로 넘어갈 때 서버에 요청하는 것이 아니라 클라이언트 내부적으로 라우터를 이용해서 서버에 요청하지 않고(데이터를 받아오지 않고)도 화면을 매끄럽게 전환할 수 있게 해준다.

* Vue는 코어 라이브러리 외에 Router 라이브러리를 공식 지원하고 있고, 아래와 같이 설치한다.

```shell
npm install vue-router [--save]
```

* Vue 라우터는 기본적으로 RootUrl'/#/'{Router name} 의 구조로 되어 있다.

```text
example.com/#/user
```

* 여기서 `#` 태그 값을 제외하고 기본 URL 방식으로 요청 때마다 index.html을 받아 라우팅을 하려면,

```js
const router = new VueRouter({
  routes,
  // 아래와 같이 history 모드를 추가해주면 된다.
  mode: 'history'
})
```

## 기본 라우터

* 뷰의 라우팅 기능을 이용하기 위해서 router-link 태그를 사용한다. 이것은 a태그로 변환된다.
* index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue Router Sample</title>
  </head>
  <body>
    <div id="app">
      <h1>Hello Vue Router!</h1>
      <p>
        <router-link to="/foo">Go to Foo</router-link>
        <router-link to="/bar">Go to Bar</router-link>
      </p>
      <router-view></router-view>
    </div>

    <script src="js/vendor/vue.js"></script>
    <script src="js/vendor/vue-router.js"></script>
    <script src="js/app.js"></script>
  </body>
</html>
```

* app.js
  * 컴포넌트를 정의하고,
  * html에서 정의한 각 라우트 경로에 컴포넌트를 맵핑한 routes를 정의한 후,
  * 뷰 라우터 인스턴스를 생성하고,
  * 뷰 루트 인스턴스를 생성하면서 라우터를 마운트 시킨다.

```js
// 0. If using a module system, call Vue.use(VueRouter)

// 1. Define route components.
// These can be imported from other files
var Foo = { template: '<div>foo</div>' }
var Bar = { template: '<div>bar</div>' }

// 2. Define some routes
// Each route should map to a component. The "component" can
// either be an actual component constructor created via
// Vue.extend(), or just a component options object.
// We'll talk about nested routes later.
var routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. Create the router instance and pass the `routes` option
// You can pass in additional options here, but let's
// keep it simple for now.
var router = new VueRouter({
  routes
})

// 4. Create and mount the root instance.
// Make sure to inject the router with the router option to make the
// whole app router-aware.
var app = new Vue({
  router
}).$mount('#app')

// Now the app has started!
```

## Nested 라우터 소개

* 라우터로 화면 이동시 Netsted(중첩된) Routers를 이용하여 여러개의 컴포넌트를 동시에 렌더링 할 수 있다.
* 렌더링 되는 컴포넌트의 구조는 가장 큰 상위의 컴포넌트가 하위의 컴포넌트를 포함하는 `Parent - Child` 형태와 같다.
* 라우터를 통해서 /home으로 이동했을 때, parent 컴포넌트 뿐만아니라 child 컴포넌트까지 같이 렌더링 한다.

```html
<!-- localhost:5000 -->
<div id="app">
  <router-view></router-view>
</div>

<!-- localhost:5000/home -->
<!-- parent component -->
<div>
  <p>Main Component rendered</p>
  <app-header></app-header>
</div>
```

## Nested 라우터 설명

* index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue Nested Router Sample</title>
  </head>
  <body>
    <div id="app">
      <h1>Hello Vue Nested Router!</h1>
      <p>
        <router-link to="/login">Go to Login</router-link>
        <router-link to="/list">Go to List</router-link>
      </p>
      <router-view></router-view>
    </div>

    <script src="js/vendor/vue.js"></script>
    <script src="js/vendor/vue-router.js"></script>
    <script src="js/app.js"></script>
  </body>
</html>
```

* app.js
  * nested 라우터의 특징은 children 속성을 정의해서 렌더링 될 하위 컴포넌트를 등록할 수 있다.
  * 이때, 추가적인 path를 설정해서 children 컴포넌트가 렌더링 되는 시점을 정할 수 있다.
  * 아래의 코드의 일부를 설명하면, /login 경로로 접근했을 때 Login 컴포넌트가 렌더링 되고, Login 컴포넌트의 \<router-view\> 태그에 children 컴포넌트가 렌더링 된다.

```js
var Login = {
  template: `
    <div>
      Login Section
      <router-view></router-view>
    </div>
  `,
};
var LoginForm = {
  template: `
    <form action="/" method="post">
      <div>
          <label for="account">E-mail : </label>
          <input type="email" id="account">
      </div>
      <div>
          <label for="password">Password : </label>
          <input type="password" id="password">
      </div>
    </form>
  `,
};
var List = {
  template: `
    <div>
      List Section
      <router-view></router-view>
    </div>
  `,
};
var ListItems = {
  template: `
    <ul>
      <li>1</li>
      <li>2</li>
      <li>3</li>
    </ul>
  `,
};

var routes = [
  {
    path: '/login',
    component: Login,
    children: [
      { path: '', component: LoginForm }
    ]
  },
  {
    path: '/list',
    component: List,
    children: [
      { path: '', component: ListItems }
    ]
  }
];

var router = new VueRouter({
  routes
});

var app = new Vue({
  router
}).$mount('#app');
```

### 주의사항 - Vue Template Root Element

* 뷰의 템플릿에는 최상위 레벨의 태그가 1개만 있어야 렌더링이 가능하다.
* 아래는 Template의 HTML 태그를 정의할 때 주의해야 하는 Vue의 성질이다.

```html
var Foo = {
  template: `
    <div>foo</div>
    <router-view></router-view>
  `  // Error compiling template 에러 발생
};
```

```text
[Vue warn]: Error compiling template:

<div>foo</div><router-view></router-view>

- Component template should contain exactly one root element. If you are using v-if on multiple elements, use v-else-if to chain them instead.
```

* 여러 개의 태그를 최상위 레벨에 동시에 위치시킬 수 없다.
* 따라서 아래와 같이 최상위 Element는 한개만 지정해야 한다.

```html
var Foo = {
  template: `
    <div>foo
      <router-view></router-view>
    </div>
};
```

## Nested 라우터 퀴즈

* index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue Nested Router Sample</title>
  </head>
  <body>
    <div id="app">
      <h1>Hello Vue Nested Router!</h1>
      <p>
        <router-link to="/login">Go to Login</router-link>
        <router-link to="/list">Go to List</router-link>
        
        <!-- 할일 #3 -->
        <!-- `/main` 에 반응할 Nested Router 를 아래에 추가하세요 -->
        <router-link to="/main">Go to Main</router-link>
      </p>
      <router-view></router-view>
    </div>

    <script src="js/vendor/vue.js"></script>
    <script src="js/vendor/vue-router.js"></script>
    <script src="js/app.js"></script>
  </body>
</html>
```

* app.js

```js
var Login = {
  template: `
    <div>
      Login Section
      <router-view></router-view>
    </div>
  `,
};
var LoginForm = {
  template: `
    <form action="/" method="post">
      <div>
          <label for="account">E-mail : </label>
          <input type="email" id="account">
      </div>
      <div>
          <label for="password">Password : </label>
          <input type="password" id="password">
      </div>
    </form>
  `,
};
var List = {
  template: `
    <div>
      List Section
      <router-view></router-view>
    </div>
  `,
};
var ListItems = {
  template: `
    <ul>
      <li>1</li>
      <li>2</li>
      <li>3</li>
    </ul>
  `,
};

// 할일 #2
// Main 컴포넌트와 그 하위 컴포넌트를 아래 등록해보세요.
var Main = {
  template: `
    <div>
      생각을 읽다, ZUM
      <router-view></router-view>
    </div>
  `,
};

var MainImg = {
  template: `
    <p>
      <img src="http://lego.zumst.com/resources/current/images/img_zum_logo_2x.png">
    </p>
  `
}

var routes = [
  {
    path: '/login',
    component: Login,
    children: [
      { path: '/login/form', component: LoginForm }
    ]
  },
  {
    path: '/list',
    component: List,
    children: [
      { path: '', component: ListItems }
    ]
  },

  // 할일 #1
  // `/main` URL 에서 동작할 라우터를 하나 등록하고,
  // 해당 라우터에서 동작할 Main 컴포넌트와 하위 컴포넌트를 생성하여 등록합니다.
  {
    path: '/main',
    component: Main,
    children: [
      { path: '', component: MainImg }
    ]
  }
];

var router = new VueRouter({
  routes
});

var app = new Vue({
  router
}).$mount('#app');
```

## Named Views 소개

* 라우터를 통해 특정 URL로 이동시, 특정 URL에 해당하는 여러개의 View(컴포넌트)를 동시에 렌더링 한다.
* 각 컴포넌트에 해당하는 `name` 속성과 `router-view` 지정 필요
* js 파일을 보면, components 속성에 값이 두개가 정의 되어있다. nestedHeader 컴포넌트에는 AppHeader에 정의된 템플릿이 오게되고, default 컴포넌트에는 Body에 정의 된 템플릿이 렌더링 된다.

```html
<div id="app">
  <router-view name="nestedHeader"></router-view>
  <router-view></router-view>
</div>
```

```js
{
  path: '/home',
  // Named Router
  components: {
    nestedHeader: AppHeader,
    default: Body
  }
},
```

## Nested Routes vs Named View?

* 특정 URL에서 1개의 컴포넌트에 여러 개의 하위 컴포넌트를 갖는 것을 Nested Routes라고 한다.
  * 하나의 부모가 다수의 하위 컴포넌트를 갖는다.
* 특정 URL에서 여러 개의 컴포넌트를 쪼개진 뷰 단위로 렌더링 하는 것을 Named View라고 한다.
  * 상위 하위의 개념이 아니라, 특정 URL로 접근했을때, 분할된 여러 개의 컴포넌트들이 한번에 렌더링 될 수 있도록 뷰에 이름을 붙인다.

![](https://github.com/namjunemy/TIL/blob/master/Vue/img/02.PNG?raw=true)