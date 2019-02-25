# Vuex

> **상태 관리 라이브러리** 
>
> Intro
>
> * 복잡한 애플리케이션의 컴포넌트들을 효율적으로 관리하는 Vuex 라이브러리 소개
> * Vuex 라이브러리의 등장 배경인 Flux 패턴(React) 소개
> * Vuex 라이브러리의 주요 속성인 **state, getters, mutations, actions** 학습
>   * Vuex를 알기 전의 개념으로는 각각 **data, computed, method, 비동기 method**라고 생각하면 된다.
> * Vuex를 더 쉽게 코딩할 수 있는 방법인 Helper 기능 소개
> * Vuex로 프로젝트를 구조화하는 방법과 모듈 구조화 방법 소개

## Vuex란?

* 무수히 많은 컴포넌트의 데이터를 관리하기 위한 상태 관리 패턴이자 라이브러리
* React의 Flux 패턴에서 기인함
* Vue.js 중고급 개발자로 성장하기 위한 필수 관문

### Flux란?

* MVC 패턴의 복잡한 데이터 흐름 문제를 해결하는 개발 패턴 - Undirectional data flow
  * 아래의 순서로 **단일 방향**으로만 흐름이 흘러간다.
    1. actions : 화면에서 발생하는 이벤트 또는 사용자의 입력
    2. dispatcher : 데이터를 변경하는 방법, 메서드(model을 바꾸기 위한 역할)
    3. model : 화면에 표시할 데이터
    4. view : 사용자에게 비춰지는 화면, 화면에서 다시 action을 호출하게 된다. 그러면 다시 1->2->3->4 한 방향 흐름.

### MVC 패턴의 문제점

* 뷰와 모델이 양방향 통신이 가능하므로 하나의 뷰가 모듈을 변경했을 때, 다시 그 모듈이 연관된 뷰들을 갱신하고, 업데이트 된 뷰들이 연관된 모델을 다시 갱신하고, 엮이고 엮이는 관계를 추적하기가 힘들다
  * ex) 페이스북 채팅화면 - 어느 사용자는 읽었고, 어느사용자는 안읽었고 이런 부분들을 처리하기 굉장히 난해했다.
* 기능 추가 및 변경에 따라 생기는 문제점을 예측할 수가 없음
* 앱이 복잡해질수록 생기는 업데이트 루프도 복잡해짐

![](https://github.com/namjunemy/TIL/blob/master/Vue/img/06.PNG?raw=true)

### Flux 패턴의 단방향 데이터 흐름

* 데이터의 흐름이 여러갈래로 나뉘지 않고 단방향으로만 처리
  * 데이터 흐름이 단방향이기 때문에 예측이 가능하다. Vue.js에서 생각하면 '상위에서 하위 뷰로 props가 내려가고, 하위에서 상위로 event가 올라가겠네?'
* 패턴 자체에서 데이터의 흐름을 정형화 시켜서 향후 발생할 수 있는 문제점들을 방지한다.

![](https://github.com/namjunemy/TIL/blob/master/Vue/img/07.PNG?raw=true)

## Vuex가 필요한 이유

* 복잡한 애플리케이션에서 컴포넌트의 개수가 많아지면 컴포넌트 간에 데이터 전달이 어려워진다.

  * Vue.js의 기본 개념을 공부했다면 알겠지만, 로그인 폼에서 id를 하위 컴포넌트로 계속해서 전달해서 내려야 할 때, 중간에 거치는 컴포넌트들이 많아 질수록 계속해서 props를 선언해야 하므로 데이터 전달이 불편해진다.

  ![](https://github.com/namjunemy/TIL/blob/master/Vue/img/08.PNG?raw=true)

  * **이벤트 버스로 해결?**

    * 어디서 이벤트를 보냈는지 혹은 어디서 이벤트를 받았는지 알기 어렵다.

      ```javascript
      // Login.vue
      eventBus.$emit('fetch', loginInfo);
      
      // List.vue
      eventBus.$on('display', data => this.displayOnScreen(data));
      
      // Chard.vue
      eventBus.$emit('refreshData', chartData);
      ```

    * 즉, 컴포넌트간 데이터 전달이 명시적이지 않다.

### Vuex로 해결할 수 있는 문제

* MVC 패턴에서 발생하는 구조적 오류
* 컴포넌트 간 데이터 전달 명시
* 여러 개의 컴포넌트에서 같은 데이터를 업데이트 할 때 동기화 문제(mutation, actions)

### Vuex 컨셉

* **State** : 컴포넌트 간에 공유하는 데이터 `data()`
* **View** : 데이터를 표시하는 화면 `template`
* **Action** : 사용자의 입력에 따라 데이터를 변경하는 `methods`

* **단방향 데이터 흐름** 처리를 단순하게 도식화한 그림

  * View(Template)에서 버튼을 클릭했을때, 클릭이라는 Action(Method)이 발생한다.
  * 해당 Action이 동작을 통해서 State(data)를 변경한다.

  ![](https://github.com/namjunemy/TIL/blob/master/Vue/img/09.PNG?raw=true)

### Vuex 구조

* 뷰 컴포넌트 -> 비동기 로직 -> 동기 로직 -> 상태
  * 시작점은 Vue Components이다.
  * 컴포넌트에서 비동기 로직(Method를 선언해서 API 콜 하는 부분 등)인 Actions를 콜하고,
  * Actions는 비동기 로직만 처리할 뿐 State(Data)를 직접 변경하진 않는다.
  * Actions가 동기 로직인 Mutations를 호출해서 State(Data)를 변경한다.
  * Mutations에서만 State(Data)를 변경할 수 있다.
* 참고자료
  * [자바스크립트 비동기 처리와 콜백 함수](https://joshua1988.github.io/web-development/javascript/javascript-asynchronous-operation/)
  * [자바스크립트 Promise 쉽게 이해하기](https://joshua1988.github.io/web-development/javascript/promise-for-beginners/)

![](https://github.com/namjunemy/TIL/blob/master/Vue/img/10.PNG?raw=true)