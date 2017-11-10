# 8. 외부 파일을 이용한 설정

우리가 어떤 스프링 프로젝트를 진행할 때, 프로젝트에 필요한 자바 코드나 설정파일이 있을텐데(예를 들면, db접근 정보, db 계정 정보, 위치 등) 이러한 정보들을 프로젝트 코드 내에 가지고 있는 것이 아니라, 프로젝트가 구동되는 시점에 외부파일에서 정보를 가져와서 사용한다. 만약, 설정 정보들이 변경되었을 경우에 프로젝트 내부 코드가 아닌 외부 설정 파일의 값만 수정함으로써 문제를 해결할 수 있다.

### Environment 객체

* Environment 객체를 이용해서 스프링 빈 설정을 한다.
* Spring Context 파일로 부터 getEnvironment() 메소드를 이용하여 Environment 객체를 얻을 수 있다.
* Environment 객체의 getPropertySources() 메소드를 이용하여 각 PropertySource 객체들(여러가지 설정에 관련된 값들)을 가져올 수 있다. 
* 즉, 아래의 구조와 같이 Spring Context 안에 설정에 관련된 값을 얻어 올 수 있는 Environment 객체가 존재하며, 이 객체 하위에는 실제 설정 값에 해당하는 PropertySource 들이 위치하게 된다.
* 실제로 어떤 정보가 필요해서 요청을 하면, Environment객체는 그 안에 있는 PropertySource들을 처음부터 끝까지 쭉 스캔해서 정보가 있는 경우 리턴한다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/Environment_01.png?raw=true)

* 위와 같이 propertySources.addLast() 메소드를 통해서 필요한 프로퍼티를 맨 마지막에 추가할 수 있고,
* env.getProperty() 메소드를 통해서 필요한 프로퍼티를 얻을 수도 있다.

### 프로퍼티 파일을 이용한 설정

### 프로파일(Profile) 속성을 이용한 설정

