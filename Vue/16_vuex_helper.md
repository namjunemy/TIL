# Vuex Helper

* Vuex의 각 속성들을 더 쉽게 사용하는 방법 - **Helper**
* Store에 있는 아래 4가지 속성들을 간편하게 코딩하는 방법
  * state -> mapState
  * getters -> mapGetters
  * mutations -> mapMutations
  * actions -> mapActions

## Helper의 사용법

* helper를 사용하고자 하는 vue 파일에서 아래와 같이 해당 helper를 로딩
* 만약 헬퍼를 사용하지 않았다면, State에 정의 된 num에 접근하려면 this.$store.state.num 와 같이 접근해야 한다.
* 생소하게 보이는 mapState앞에 있는 `...`은 ES6의 Object Spread Operator이다.
  * [Vue.js를 위한 ES6 정리 내용 링크](https://github.com/namjunemy/TIL/blob/master/Vue/es6_for_vuejs.md)

```javascript
//App.vue
import { mapState } from 'vuex';
import { mapGetters } from 'vuex';
import { mapMutations } from 'vuex';
import { mapActions } from 'vuex';

export default {
  computed() {
    ...mapState(['num']),
    ...mapGetters(['countedNum'])
  },
  methods: {
    ...mapMutations(['clickBtn']),
    ...mapActions(['asyncClickBtn'])
  }
}
```

## mapState, mapGetters

* mapState

  * vuex에 선언한 state 속성을 뷰 컴포넌트에 더 쉽게 연결해주는 헬퍼

  * vuex에서 mapState 함수를 가져와서 ... 연산자로 연결하고, 배열리터럴로 확장해서 사용한다.

  * vuex store에 선언된 state에 this.$store.state로 접근하지 않아도 됨

  * 또, vue 템플릿에서 state안에있는 num 접근할 때도 this.num으로 바로 접근 가능하다.

    ```javascript
    // App.vue
    import { mapState } from 'vuex';
    
    computed() {
      ...mapState(['num'])
      // num() { return this.$store.state.num; } 를 대체함
    }
    
    // store.js
    state: {
      num: 10
    }
    ```

    ```vue
    <!-- <p>{{ this.$store.state.num }}</p> -->
    <p>{{ this.num }}</p>
    ```

* mapGetters

  * vuex에 선언한 getters 속성을 뷰 컴포넌트에 더 쉽게 연결해주는 헬퍼

  * vuex store에 선언된 getters에 this.$store.getters로 접근하지 않아도 됨

  * getters는 vuex를 적용하기 전 사용하던 computed 속성이라고 생각하면 된다.

    ```javascript
    // App.vue
    import { mapGetters } from 'vuex';
    
    computed: {
      ...mapGetters(['reverseMessage'])
    }
    
    // store.js
    getters: {
      reverseMessage(state) {
        return state.msg.split('').reverse().join('');
      }
    }
    ```

    ```vue
    <!-- <p>{{ this.$store.getters.reverseMessage }}</p> -->
    <p>{{ this.reverseMessage }}</p>
    ```

### ES6 Spread 연산자를 쓰는 이유

* 싱글파일 컴포넌트 구조의 각 컴포넌트에서 정의된 computed 속성이 있을 것이고,
* 거기에 Vuex store에 정의된 getters를 사용해야하는 상황이 있을 것이다.
* 이때, `...` 연산자를 통해 mapGetters를 연결해줘야 컴포넌트에서 computed 속성과 Vuex의 store에 등록된 getters를 함께 사용할 수 있다.

## mapMutations, mapActions

* mapMutations

  * Vuex에 선언한 mutations 속성을 뷰 컴포넌트에 더 쉽게 연결해주는 헬퍼

    * 컴포넌트에서 버튼 클릭시 컴포넌트의 methods에 clickBtn이 정의 되어있는 것 처럼 동작한다.

    * 하지만 실제로는 ...mapMutations를 통해서 Vuex Store에 등록된 함수를 쓴다.

    * 따라서, Vuex에 정의된 state.msg가 alert로 뜨게 될 것이다.

      ```javascript
      // App.vue
      import { mapMutations } from 'vuex';
      
      methods: {
        ...mapMutations(['clickBtn']),
        authLogin() {},
        displayTable() {}
      }
      
      // store.js
      mutations: {
        clickBtn(state) {
          alert(state.msg);
        }
      }
      ```

      ```vue
      <button @click="clickBtn">popup message</button>
      ```

* mapActions

  * Vuex에 선언한 actions 속성을 뷰 컴포넌트에 더 쉽게 연결해주는 헬퍼

    * 마찬가지로 컴포넌트에서 버튼 클릭으로 methods에 등록된 delayClickBtn을 호출한다.

    * methods에는 Spread Operator로 store의 actions에 정의된 delayClickBtn을 등록한다.

      ```javascript
      // App.vue
      import { mapActions } from 'vuex';
      
      methods: {
        ...mapActions(['delayClickBtn'])
      }
      
      // store.js
      actions: {
        delayclickBtn(context) {
          setTimeout(() => context.commit('clickBtn'), 2000);
        }
      }
      ```

      ```vue
      <button @click="delayClickBtn">delay popup message</button>
      ```

## Helper의 유연한 문법

* Vuex의 선언한 속성을 그대로 컴포넌트에 연결하는 문법

  * addNumber(인자)의 경우 배열리터럴로 넘기는 string에 선언을 안해도 알아서 넘겨준다!

  ```javascript
  // 배열 리터럴
  ...mapMutations([
    'clickBtn',    //'clickBtn': clickBtn
    'addNumber'    //addNumber(인자)
  ])
  ```

* Vuex에 선언한 속성을 컴포넌트의 특정 메서드에다가 연결하는 문법

  * 컴포넌트에서 사용하는 메서드와 Store의 Mutation 명이 다를 경우 객체 리터럴을 사용해서 등록할 수 있다.

  ```javascript
  // 객체 리터럴
  ...mapMutations({
    popupMsg: 'clickBtn'    // 컴포넌트 메서드 명 : Store의 Mutation 명
  })
  ```


## Helper 함수가 주는  간편함

* demoStore.js

  ```javascript
  import Vue from 'vue';
  import Vuex from 'vuex';
  
  Vue.use(Vuex);
  
  export const store = new Vuex.store({
    state: {
      price: 100
    },
    getters: {
      originalPrice(state) {
        return state.price;
      },
      doublePrice(state) {
        return state.price * 2;
      },
      triplePrice(state) {
        return state.price * 3;
      }
    }
  })
  ```

* 헬퍼를 사용하지 않고 Vuex를 적용한 컴포넌트 - Demo.vue

  * template영역에 this.$store로 스토어에 바로 접근해도 되지만, Vue는 template 영역에 표현식을 줄이는 것을 권고한다. 따라서 스크립트 영역에서 this.$store로 값을 받아온다. 

  ```vue
  <template>
    <div id="root">
      <p>{{ originalPrice }}</p>  //100
      <p>{{ doublePrice }}</p>    //200
      <p>{{ triplePrice }}</p>    //300
    </div>
  </template>
  
  <script>
    export default {
      computed: {
        originalPrice() {
          return this.$store.getters.originalPrice;
        },
        doublePrice() {
          return this.$store.getters.doublePrice;
        },
        triplePrice() {
          return this.$store.getters.triplePrice;
        },
      },
    };
  </script>
  ```

  * Vuex 의 헬퍼함수를 사용하면 반복 코드를 줄이고 편하게 Vuex store에 접근할 수 있다.

    ```vue
    <template>
      <div id="root">
        <p>{{ originalPrice }}</p>
        <p>{{ doublePrice }}</p>
        <p>{{ triplePrice }}</p>
      </div>
    </template>
    
    <script>
      import {mapGetters} from 'vuex';
    
      export default {
        computed: {
          ...mapGetters(['originalPrice', 'doublePrice', 'triplePrice']),
        },
      };
    </script>
    ```

## Vuex 프로젝트 구조화 및 모듈화

* 프로젝트 구조화와 모듈화 방법 1

  * 속성별 파일로 구조화

  * store 디렉토리 구조

    * store
      * getters.js
      * mutations.js
      * store.js

  * getters.js

    ```javascript
    export const storedTodoItems = (state) => {
      return state.todoItems;
    }
    ```

  * mutations.js

    ```javascript
    const addOneItem = (state, todoItem) => {
      const obj = {completed: false, item: todoItem};
      sessionStorage.setItem(todoItem, JSON.stringify(obj));
      state.todoItems.push(obj);
    }
    
    const removeOneItem = (state, payload) => {
      sessionStorage.removeItem(payload.todoItem.item);
      state.todoItems.splice(payload.index, 1);
    }
    
    const toggleOneItem = (state, payload) => {
      state.todoItems[payload.index].completed = !state.todoItems[payload.index].completed;
      sessionStorage.removeItem(payload.todoItem.item);
      sessionStorage.setItem(payload.todoItem.item,
                             JSON.stringify(payload.todoItem));
    }
    
    const clearAll = (state) => {
      state.todoItems = [];
      sessionStorage.clear();
    }
    
    export { addOneItem, removeOneItem, toggleOneItem, clearAll }
    ```

  * store.js

    ```javascript
    import Vue from 'vue';
    import Vuex from 'vuex';
    import * as getters from './getters';
    import * as mutations from './mutations'
    
    Vue.use(Vuex);
    
    export const store = new Vuex.Store({
      state: {
        todoItems: storage.fetch()
      },
      getters,
      mutations
    })
    ```

* **방법 2 - 앱이 비대해져서 1개의 store로는 관리가 힘들 때 `modules` 속성 사용**

  * 모듈별로 필요한 store 속성들을 모아서 관리한다.

  * store.js

    ```javascript
    import Vue from 'vue';
    import Vuex from 'vuex';
    import todoApp from './modules/todoApp';
    
    Vue.use(Vuex);
    
    export const store = new Vuex.Store({
      modules: {
        todoApp
      }
    });
    ```

  * store/modules/todoApp.js

    ```javascript
    const storage = {
      fetch() {
        const arr = [];
        if (sessionStorage.length > 0) {
          for (let i = 0; i < sessionStorage.length; i++) {
            if (sessionStorage.key(i) !== 'loglevel:webpack-dev-server') {
              arr.push(JSON.parse(sessionStorage.getItem(sessionStorage.key(i))));
            }
          }
        }
        return arr;
      },
    };
    
    const state = {
      todoItems: storage.fetch(),
    }
    
    const getters = {
      storedTodoItems(state) {
        return state.todoItems;
      }
    }
    
    const mutations = {
      addOneItem(state, todoItem) {
        const obj = {completed: false, item: todoItem};
        sessionStorage.setItem(todoItem, JSON.stringify(obj));
        state.todoItems.push(obj);
      },
      removeOneItem(state, payload) {
        sessionStorage.removeItem(payload.todoItem.item);
        state.todoItems.splice(payload.index, 1);
      },
      toggleOneItem(state, payload) {
        state.todoItems[payload.index].completed = !state.todoItems[payload.index].completed;
        sessionStorage.removeItem(payload.todoItem.item);
        sessionStorage.setItem(payload.todoItem.item,
                               JSON.stringify(payload.todoItem));
      },
      clearAll(state) {
        state.todoItems = [];
        sessionStorage.clear();
      }
    }
    
    export default { state, gettets, mutations }
    ```

    