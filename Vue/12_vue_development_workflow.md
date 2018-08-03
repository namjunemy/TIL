# Vue Development Workflow

* vue cli 로 간단한 webpack 설정이 되어 있는 프로젝트 생성이 가능하다.

```shell
npm install -global vue-cli
vue init webpack-simple
npm install
npm run dev
```

```js
export default {
  // 이 안의 내용은 모두 Vue Instance에 포함되어 생성된다.
}
```

## Vue CLI를 이용한 프로젝트 구성 팁

* vue cli를 이용해서 프로젝트 구성을 할 때,
* vue cli를 heavy 하게 사용해보지 않은 이상 webpack-simple을 활용해서 프로젝트를 구성하는 것을 추천한다.

## Vue CLI와 싱글 파일 컴포넌트를 이용한 컴포넌트 등록

* webpack-simple로 구성한 프로젝트 기본구조

![](https://github.com/namjunemy/TIL/blob/master/Vue/img/04.PNG?raw=true)

* 싱글 파일 컴포넌트구조로 .vue 파일에 커스텀 컴포넌트를 추가한 프로젝트 구조

![](https://github.com/namjunemy/TIL/blob/master/Vue/img/05.PNG?raw=true)

* App.vue 파일

```vue
<template>
  <div id="app">
    <!-- Global Component, check main.js -->
    <nav-header></nav-header>
    // 할일 #2
    <global-h1></global-h1>
    <login-form></login-form>
    // 할일 #1
    <local-h1></local-h1>
  </div>
</template>

<script>
  import LoginForm from './components/LoginForm.vue';
  import LocalH1 from './components/LocalH1.vue';

  export default {
    components: {
      'login-form': LoginForm,
      // 할일 #1
      // Local 컴포넌트를 하나 등록하고, login form 아래에 렌더링 해보세요.
      'local-h1': LocalH1
    }
  }
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
```

* main.js 파일

```js
import Vue from 'vue'
import App from './App.vue'
// Global Components
import NavHeader from './components/NavHeader.vue';

Vue.component('nav-header', NavHeader);
// 할일 #2
// Global 컴포넌트를 하나 더 등록하고 nav-header 아래에 렌더링 해보세요.
import GlobalH1 from './components/GlobalH1.vue'

Vue.component('global-h1', GlobalH1);

new Vue({
  el: '#app',
  render: h => h(App)
})
```

* GlobalH1.vue 파일

```vue
<template lang="html">
  <div>
    <h1>Global H1</h1>
  </div>
</template>
```

* LovalH1.vue 파일

```vue
<template lang="html">
  <div>
    <h1>Local H1</h1>
  </div>
</template>
```

