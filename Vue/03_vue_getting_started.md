# Vue 시작하기

> Inflearn - Captain Pangyo

## Vue는 무엇인가?

* MVVM 패턴의 ViewModel 레이어에 해당하는 View 단 라이브러리
  * MVVM(Model-View-ViewModel)
  * 모델과 뷰 사이에 뷰모델이 위치하는 구조
  * DOM이 변경됐을 때, 뷰모델의 DOM Listeners를 거쳐서 모델로 신호가 간다.
  * 모델에서 변경된 데이터를 뷰모델을 거쳐서 뷰로 보냈을 때, 화면이 reactive하게 반응이 일어난다.
  * Vue.js 는 이와 같이 reactive한 프로그래밍이 가능하게 끔 뷰단에서 모든 것을 제어하는 뷰모델 라이브러리이다.

![](https://github.com/namjunemy/TIL/blob/master/Vue/img/01.PNG?raw=true)



* 데이터 바인딩과 화면 단위를 컴포넌트 형태로 제공하며, 관련 API를 지원하는데에 궁극적인 목적이 있음
* Angular에서 지원하는 2 way data bindings을 동일하게 제공
* 하지만 Component 간 통신의 기본 골격은 React의 1way data flow(부모 -> 자식)와 유사
* Virtual DOM을 이용한 렌더링 방식이 React와 거의 유사
* 다른 Front-End FW(Angular, React)와 비교했을 때 훨씬 가볍고 빠름
* 간단한 Vue를 적용하는데 있어서도 러닝커브가 낮고, 쉽게 접근 가능

> 위의 내용들은 강좌가 진행 되면서 자세히 알게 될 것

## Hello Vue.js

* index.html

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
    new Vue({
      el: '#app',
      data: {
        message: 'Hello Vue.js!!'
      }
    })
  </script>
</body>
</html>
```

## MVVM 패턴이란?

Backend 로직과 Client의 마크업 & 데이터 표현단을 분리하기 위한 구조로 전통적인 MVC 패턴 방식에 기인하였다. 간단하게 생각해서 화면 앞단의 화면 동작 관련 로직과 뒷단의 DB 데이터 처리 및 서버 로직을 분리하고, 뒷단에서 넘어온 데이터를 Model에 담아 View로 넘겨주는 중간 지점이라고 보면 된다.