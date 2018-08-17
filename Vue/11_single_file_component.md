# Single File Components with JSX(ES6)

> 누구나 다루기 쉬운 Vue.js 프론트 개발 - Captain Pangyo [링크](https://www.inflearn.com/course/vue-pwa-vue-js-%EA%B8%B0%EB%B3%B8/)

* 앱의 복잡도가 증가할 때, `.vue` 라는 파일 단위 안에 html, js, css 를 관리할 수 있는 방법
* 복잡도가 커짐에 따라 야기될 수 있는 문제들
  * 모든 컴포넌트에 고유의 이름을 붙여야 함
  * js 파일에서 template 안의 html의 문법 강조가 되지 않음
  * js 파일 상에서 css 스타일링 작업이 거의불가
  * ES5를 이용하여 계속 앱을 작성할 경우 Babel 빌드가 지원되지 않음
* `.vue` 파일을 브라우저가 렌더할 수 있는 파일들로 변환하려면 webpack의 vue-loader 또는 browserify 이용

```html
<template>
  <!-- -->
</template>

<script>
  <!-- -->
</script>

<style>
  <!-- -->
</style>
```

> 참고 : ES5 만 사용하는 경우 single file component의 혜택을 볼 수 없음

