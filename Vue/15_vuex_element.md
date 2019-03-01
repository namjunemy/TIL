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

### state

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

### getters

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

### mutations

* state 값을 변경하는 이벤트 로직, 메서드 `methods`

* 뮤테이션은 `commit()`으로 동작시킨다.

* 뮤테이션은 항상 첫번째 인자로 state를 가져온다. vue의 규약이라고 보면 된다.

  ```javascript
  // store.js
  state: { num: 10 },
  mutations: {
    printNumbers(state) {
      return state.num
    },
    sumNumbers(state, anotherNum) {
      return state.num + anotherNum;
    }
  }
  
  // App.vue
  this.$store.commit('pringNumbers');    //10
  this.$store.commit('sumNumbers', 20);  //30
  ```

* mutations의 commit() 형식

  * state를 변경하기 위해 mutations를 동작시킬 때 인자(payload)를 전달할 수 있음

    ```javascript
    // store.js
    state: {
      storeNum:10
    },
    mutations: {
      modifyState(state, payload) {
        console.log(payload.str);
        return state.storeNum += payload.num;
      }
    }
    
    // App.vue
    this.$store.commit('modifyState', {
      str: 'passed from payload',
      num: 20
    });
    ```

* **왜 mutations로 상태를 변경해야 하는가?**

  * 여러개의 컴포넌트에서 아래와 같이 state 값을 변경하는 경우 어느 컴포넌트에서 해당 state를 변경했는지 추적하기 어렵다.

    ```javascript
    methods: {
      increaseCounter() { this.$store.state.counter++; }
    }
    ```

  * 특정 시점에 어떤 컴포넌트가 state를 접근하여 변경한 건지 확인하기 어렵기 때문이다.

  * 따라서, 뷰의 반응성(화면의 변화를 스크립트에서 인지하는것 등)을 거스르지 않게 명시적으로 상태변화를 수행한다. 반응성, 디버깅, 테스팅 혜택

### actions

* 비동기 처리 로직을 선언하는 메서드 `async methods`. 즉, 비동기 로직을 담당하는 mutations이다.

* 데이터 요청, Promise, ES6 async과 같은 비동기 처리는 모두 actions에 선언한다.

* 컴포넌트에서 `#store.dispatch()`로 actions 호출, 비동기 로직 수행 후 `context.commit()`으로 vuex store의 mutations 호출.

  ```javascript
  // store.js
  state: {
    num: 10
  },
  mutations: {
    doubleNumber(state) {
      state.num * 2;
    }
  },
  actions: {
    delayDoubleNumber(context) {     //context로 store의 메서드와 속성 접근
      context.commit('doubleNumber');
    }
  }
  
  // App.vue
  this.$store.dispatch('delayDoubleNumber');
  ```

* actions 비동기 코드 예제 1

  * Component의 methods에서 vuex actions 비동기 메서드 호출 -> 2초 후 vuex mutations 동기 메서드 수행

  ```javascript
  // store.js
  mutations: {
    addCounter(state) {
      state++;
    }
  },
  actions: {
    delayedAddCounter(context) {     //context로 store의 메서드와 속성 접근
      setTimeout(() => context.commit('addCounter'), 2000);
    }
  }
  
  // App.vue
  methods: {
    incrementCounter() {
      this.$store.dispatch('delayedAddCounter');
    }
  }
  ```

* actions 비동기 코드 예제 2

  ```javascript
  // store.js
  mutations: {
    setData(state, fetchedData) {
      state.product = fetchedData;
    }
  },
  actions: {
    fetchedProductData(context) {
      return axios.get('http://domain.com/products/1')
                  .then(response => context.commit('setData', response));
    }
  }
  
  // App.vue
  methods: {
    getProduct() {
      this.$store.dispatch('fetchedProductData');
    }
  }
  ```

* **왜 비동기 처리 로직은 actions에 선언해야 할까?**

  * 언제 어느 컴포넌트에서 해당 state를 호출하고, 변경했는지 확인하기가 어려움

  * 여러개의 컴포넌트에서 mutations로 시간차를 두고 state를 변경하는 경우

    * state값의 변화를 추적하기 어렵기 때문에 mutations 속성에는 처리 시점을 예측할 수 있는 동기 처리 로직만 넣어야 한다.

      ![](https://github.com/namjunemy/TIL/blob/master/Vue/img/11.PNG?raw=true)

## Reference

- https://vuejs.org/v2/guide/
- https://kr.vuejs.org/v2/guide/
- https://www.inflearn.com/course/vue-pwa-vue-js-%EC%A4%91%EA%B8%89/