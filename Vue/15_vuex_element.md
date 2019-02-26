# Vuex 주요 기술 요소

## Install

* Vuex는 싱글 파일 컴포넌트 체계에서 NPM 방식으로 라이브러리를 설치하는게 좋다.

```shell
npm install vuex --save
```

* ES6와 함께 사용해야 더 많은 기능과 이점을 제공받을 수 있음

* Vue.js에서 전역으로 Vuex 플러그인 사용하기

  * /src/store/store.js

    ```javascript
    import Vue from 'vue';
    import Vuex from 'vuex';
    
    Vue.use(Vuex);
    export const store = new Vuex.Store({
      //
    });
    ```

  * /src/main.js

    * export한 store를 import 시키고
    * ES6의 Enhanced Object Literals 특성을 통해 축약해서 등록할 수 있다.

    ```javascript
    import Vue from 'vue';
    import App from './App.vue';
    import { store } from './store/store'
    
    new Vue({
      el: '#app',
      store,	              // store: store,
      render: h => h(App),
    });
    ```


## Vuex 기술 요소

* state

  * 여러 컴포넌트에 공유되는 데이터 `data`

  ```javascript
  // Vue
  data: {
    message: 'Hello Vue.js'
  }
  
  // Vuex
  state: {
    message: 'Hello Vue.js'
  }
  ```

  ```vue
  <!-- Vue -->
  <p>{{ message }}</p>
  
  <!-- Vuex -->
  <p>{{ this.$store.state.message }}</p>
  ```

* getters

  * 연산된 state 값을 접근하는 속성 `computed`
  * 뒤에서 헬퍼함수를 통해 `this.$store.state.message`로 state를 접근하는 법을 간소화함

  ```javascript
  // store.js
  state: {
    num: 10
  },
  getters: {
    getNumber(state) {
      return state.num;
    },
    doubleNumber(state) {
      return state.num * 2;    
    }
  }
  ```

  ```vue
  <p>{{ this.$store.getters.getNumber }}</p>
  <p>{{ this.$store.getters.doubleNumber }}</p>
  ```

* mutations

  * state 값을 변경하는 이벤트 로직, 메서드 `methods`

* actions

  * 비동기 처리 로직을 선언하는 메서드 `async methods`



