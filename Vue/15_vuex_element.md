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

    

