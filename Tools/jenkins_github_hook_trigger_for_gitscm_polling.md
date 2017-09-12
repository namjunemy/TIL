# Github Webhook을 이용한 자동 빌드 환경 구축

Github 저장소와 젠킨스 프로젝트를 연동을 통해 빌드 환경을 구성 한 후, Github Webhook을 이용하여 Github 저장소에 push된 것이 있다면 그 때 젠킨스가 polling하여 빌드를 실행하는 자동 빌드 환경을 구축한다.

*[블로그 포스팅](http://ict-nroo.tistory.com/37)을 참고하여 화면 설명과 함께 환경을 구축하는 것을 추천합니다:)*



### Github프로젝트 Jenkins 서비스 추가

* Github프로젝트의 Settings로 진입하여 왼쪽 메뉴의 Integrations & services 탭으로 진입하고, Add service 버튼을 클릭하여 jenkins 키워드로 검색한다. Jenkins(Github plugin)를 선택한다.
* Jenkins hook url을 추가하는 란에 "http://{본인의 ip}:8080/jenkins/github-webhook/" 을 입력하고, Active에 체크 한 후 서비스를 추가한다.



### Github프로젝트 Webhook 추가

* 마찬가지로 Github프로젝트의 Setting에서 Webhooks 탭으로 진입한다. Add webhook 버튼을 누른다.

* Payload URL에 "http://{본인의 ip}:8080/jenkins/github-webhook/" 입력하고, Content type을 선택한다. 아래의 Just the push event에 체크한 이유는 Github repo에 push 이벤트가 일어날 경우 이 웹훅을 유발시키기 위함이다. webhook을 추가한다.


### 젠킨스 프로젝트 구성의 Build Trigger 설정 

* 젠킨스 프로젝트의 구성 탭으로 이동하여, GITScm polling을 위한 Github hook trigger를 빌드 유발 설정으로 선택한다. 이 선택은 위의 SCM(소스 코드 관리) 탭에서 연동한 Github 저장소에서 push에 의한 hook 이벤트가 발생할 경우 저장소를 polling해서 젠킨스의 자동 빌드를 유발한다. 







