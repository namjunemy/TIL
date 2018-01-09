## casperjs로 크롤링하기

casperjs는 phantomjs를 포함하고 있다. 정확히 말하면 head less application인 phantomjs를 좀 더 쉽고 편하게 사용하기 위한 기능들을 모아놓은 라이브러리이다. 웹의 네비게이션을 자동화하여 필요한 데이터를 추출할 수 있다. 예를 들면, 로그인을 한 뒤 어떤 정보를 가져오는 것처럼 사람의 행동이 필요한 작업들을 phantomjs는 해낼 수 있다. 실제로 phantomjs는 웹 브라우저 엔진(웹킷. 애플의 웹 프라우저인 사파리에서 사용하는 렌더링 엔진)을 가지고 있고, 이 것은 사람이 브라우저를 열어서 웹 Documents를 해석해서 이벤트를 주는 것과 동일한 일을 할 수 있다는 것을 의미한다. phantomjs를 이용하면 CLI로 브라우저 조작, 데이터 취득, 스크린샷 등이 가능해지며 웹사이트에서 데이터를 스크래핑 또는 크롤링하거나 UI테스트 자동화 등에도 쓰일 수 있다. casperjs는 phanromjs를 쉽게 사용하기 위한 라이브러리 이므로, phantomjs의 사전 설치가 필요하다.

> windows 10

* casperjs 설치
* npm global로 phantomjs, casperjs 설치 후 환경변수 설정
* npm으로 설치하지만 casperjs는 node 모듈이 아님. npm 으로 편하게 설치하는 것 뿐임.
* python2.7설치
* phantomjs 버전에 따라 es6 지원 여부 결정
* 실행은 casperjs [filename].js

### 예제 코드

* 스크린 샷 생성

```javascript
var links = [];
var casper = require('casper').create({
  viewportSize: {
    width: 1024,
    height: 768
  }
});

function getLinks() {
  var links = document.querySelectorAll('h3.r a');
  return Array.prototype.map.call(links, function (e) {
    return e.getAttribute('href');
  });
}

casper.start('http://google.fr/', function () {
  var fileLocate = 'screenShotTest/1.jpg';
  this.captureSelector(fileLocate, "html");
  this.fill('form[action="/search"]', {q: 'seotory'}, true);
});

casper.then(function () {
  var fileLocate = 'screenShotTest/2.jpg';
  this.captureSelector(fileLocate, "html");
  links = this.evaluate(getLinks);
  this.fill('form[action="/search"]', {q: 'wordpress'}, true);
});

casper.then(function () {
  var fileLocate = 'screenShotTest/3.jpg';
  this.captureSelector(fileLocate, "html");
  links = links.concat(this.evaluate(getLinks));
});

casper.run(function () {
  this.echo(links.length + ' links found:');
  this.echo(' - ' + links.join('\n - ')).exit();
});
```

* 데이터 크롤링

```javascript
var casper = require('casper').create({
  verbose: true,
  logLevel: "debug",
  viewportSize: {
    width: 1024,
    height: 768
  }
});

casper.start('https://company.lotte.net/', function () {
  this.fill('form[action="/?ReturnUrl=https%3A%2F%2Fcompany.lotte.net%3A443%2F"]', {
    LoginId: 'USER_ID',
    Password: 'USER_PASSWORD'
  }, true);
});

casper.then(function () {
  this.echo("### 접속 확인");
});

casper.then(function () {
  var fileLocate = 'screenShotTest/loginTest.jpg';
  this.captureSelector(fileLocate, "html");
  this.echo("### 스크린샷 생성");
});

casper.then(function () {
  var name = function () {
    return document.querySelector('.name > em').innerHTML;
  };
  var team = function () {
    return document.querySelector('.team').innerHTML;
  };
  console.log("### 접속자: " + this.evaluate(team) + " " + this.evaluate(name));
});

casper.thenOpen('https://common.lotte.net/Organization/AjaxOrgTreeJson', function () {
  var jsonData = function () {
    return document.querySelector('body').innerHTML;
  };
  var resultArray = JSON.parse(this.evaluate(jsonData) + '');
  for (var i = 0; i < resultArray.length; i++) {
    console.log(resultArray[i].text);
    if (resultArray[i].children != null) {
      for (var j = 0; j < resultArray[i].children.length; j++) {
        console.log('  ' + resultArray[i].children[j].text);
      }
    }
  }
});

casper.run();
```

> 참고
>
> [https://junistory.blogspot.kr/2017/08/phantomjs-casperjs.html](https://junistory.blogspot.kr/2017/08/phantomjs-casperjs.html)
>
> [http://blog.saltfactory.net/web-scraping-using-with-casperjs/](http://blog.saltfactory.net/web-scraping-using-with-casperjs/)
>
> [http://harrythegreat.tistory.com/entry/phantomjs-CapserJS%EB%A5%BC-%ED%86%B5%ED%95%9C-%ED%81%AC%EB%A1%A4%EB%A7%81-%EC%A0%95%EB%A6%AC-1](http://harrythegreat.tistory.com/entry/phantomjs-CapserJS%EB%A5%BC-%ED%86%B5%ED%95%9C-%ED%81%AC%EB%A1%A4%EB%A7%81-%EC%A0%95%EB%A6%AC-1)
>
> [http://forum.falinux.com/zbxe/?mid=lecture_tip&page=2&l=mn&document_srl=796325](http://forum.falinux.com/zbxe/?mid=lecture_tip&page=2&l=mn&document_srl=796325)
>
> 

