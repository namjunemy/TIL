# Vue Resource

> 누구나 다루기 쉬운 Vue.js 프론트 개발(인프런) - Captain Pangyo [링크](https://www.inflearn.com/course/vue-pwa-vue-js-%EA%B8%B0%EB%B3%B8/)

Vue에서 HTTP 통신을 위해 제공하는 플러그인

* es6에서는 아래와 같이 다운로드, 웹팩을 사용하지 않는 경우에는 뷰 리소스 cdn을 통해서 실행하면 된다.
* 주의할 점은 현재 공식 지원은 중단했다. 이유는 라이브러리에 제한을 두지 않고 사용자들의 선택권을 넓히기 위함이라고 한다.

```shell
npm install vue-resource --save
```

* 위의 명령어로 설치 후 Root Vue Instance를 선언하는 js파일에 아래와 같이 등록

```js
// 후에 배울 Single File Component 구조 기준
import VueResource from 'vue-resource';
...
Vue.use(VueResource);
```

* 사용법은 아래와 같다.

```js
this.$http.get(url).then(successCallback, failCallback);
```

### axios

* 브라우저와 node를 위한 프로미스 기반의 http 클라이언트이고, 요즘 가장 많이 쓰는 라이브러리이다.

