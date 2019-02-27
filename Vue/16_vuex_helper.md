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

