# Vue Loader

> 누구나 다루기 쉬운 Vue.js 프론트 개발(인프런) - Captain Pangyo [링크](https://www.inflearn.com/course/vue-pwa-vue-js-%EA%B8%B0%EB%B3%B8/)

* `.vue` 형태의 파일을 javascript 모듈 형태로 변환해주는 webpack loader
  * .vue 파일을 인식해서 최종적으로 페이지에 뿌릴 수 있게 변환해준다.
* 프로젝트 구조에서

![](https://github.com/namjunemy/TIL/blob/master/Vue/img/05.PNG?raw=true)

* index.html의 내용을 보면, `/dist/build.js` 를 참조하는 스크립트가 보인다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, user-scalable=no">
    <title>single-file-components</title>
  </head>
  <body>
    <div id="app"></div>
    <script src="/dist/build.js"></script>
  </body>
</html>
```

* Vue Loader를 통해서 변환된 .vue 파일의 내용이 build.js를 통해서 화면에 그려지게 된다.
* Vue Loader로 인해 얻게 되는 장점들은
  * ES6 지원
    * babel loader, import from 구문
  * `<style>` 과 `<template>` 에 대한 각각의 webpack loader 지원. ex) sass. jade
  * 각 `.vue` 컴포넌트의 스코프로 좁힌 css 스타일링 지원
    * scoped 속성을 추가하면 스타일 자체는 컴포넌트에만 적용 된다.
  * webpack의 모듈 번들링에 대한 지원과 의존성 관리가 제공
  * 개발시 hot reloading 지원
    * webpack dev server를 지원하기 때문에, 개발할 때 새로고침 하지 않아도 바로바로 화면에 적용된다.

## 정리

* Vue Instance
  * 화면을 개발하기 위해 필요한 단위, 인스턴스가 있어야만 화면을 렌더링 했을 때(웹 페이지에 뷰의 코드들이 올라갔을 때) 우리가 볼 수 있는 가시적인 형태로 나타난다.
* Vue Components
  * 페이지의 구성 요소가 하나하나의 컴포넌트가 된다. 화면을 단위별로 쪼개는 것이 컴포넌트의 개념이다.
* Vue Router
  * 화면전환을 위해서 라우터의 개념이 필요하다. SPA를 개발하기 위해서는 라우터는 거의 필수적이다.
* Vue Template, Vue Data Binding
  * 화면의 요소에 들어가는 값을 찍어내기 위해서 하드코딩으로 넣을 수도 있지만, 값들이 서버에 있다면, 그것들을 맵팽해주는 작업을 템플릿을 통해서 한다. 템플릿을 통해 해당 데이터를 맵핑시키는 Data Binding까지 한다.
* Vue Resource
  * 리소스 역시 서버에 있는 데이터를 가져오기 위해, http 요청을 지원하는 라이브러리로 axios나 vue resource가 있다.
* Vue Single File Components
  * 실제로 뷰로 개발을 진행할 때에는 Single File Component로 vue cli를 이용해서 프로젝트를 구성해야만, 작업 효율과 가독성을 확보할 수 있다.
* Vue Loader
  * Single File Component로 프로젝트를 구성하는데 있어서 .vue 파일이 인식될 수 있게하는 뷰 로더가 필요하고, 여기서 웹팩이 필요하다.

### Advanced 레벨을 위한 학습 커리큘럼 제안

* Vue Reactivity System
* Render Function
* State Management
* Server side Rendering

