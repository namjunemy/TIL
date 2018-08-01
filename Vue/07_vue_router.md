# Vue Routers

> Inflearn - Captain Pangyo

## Intro

일반적으로 뷰에서 화면이 전환될 때, 전환하는 행위를 Route라고 표현한다. 뷰에서는 SPA를 제작할 때 유용한 라우팅 라이브러리로 Vue Routers를 제공하고 있다.

> **SPA에 대하여**
>
> 일반적으로 웹 어플리케이션은 해당 웹 페이지를 서버에 요청하고, 서버가 html을 돌려 줬을 때 그것을 랜더링 엔진을 통해 화면에 보여준다. SPA는 해당 화면에 대한 정보를 미리 가지고 있다가, 해당 화면으로 넘어갈 때 서버에 요청하는 것이 아니라 클라이언트 내부적으로 라우터를 이용해서 서버에 요청하지 않고(데이터를 받아오지 않고)도 화면을 매끄럽게 전환할 수 있게 해준다.

* Vue는 코어 라이브러리 외에 Router 라이브러리를 공식 지원하고 있고, 아래와 같이 설치한다.

```shell
npm install vue-router [--save]
```

* Vue 라우터는 기본적으로 RootUrl'/#/'{Router name} 의 구조로 되어 있다.

```text
example.com/#/user
```

* 여기서 `#` 태그 값을 제외하고 기본 URL 방식으로 요청 때마다 index.html을 받아 라우팅을 하려면,

```js
const router = new VueRouter({
  routes,
  // 아래와 같이 history 모드를 추가해주면 된다.
  mode: 'history'
})
```

