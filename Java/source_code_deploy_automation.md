# 소스코드 배포 과정에 대한 자동화

### 배포과정

* 첫번째 배포
  * git clone "GIT REPO URL"
  * cd PROJECT_DIRECTORY
  * mvn clean package
  * TOMCAT_PATH/libexec/bin/shutdown.sh
  * rsync -avr —delete "PROJECT_DERECTORY/target/CONTEXT_NAME" "TOMCAT_PATH/wepapps/"
    * Ex) rsync -avr —delete /Users/njkim/Workspace/github/SLiPP/apps/servlet_jsp_project_SLiPP/target/qna /usr/local/Cellar/tomcat/9.0.5/libexec/webapps
  * TOMCAT_PATH/libexec/bin/startup.sh
* 수정사항 반영 후 배포과정
  * git pull [origin master]
  * mvn clean package 
  * TOMCAT_PATH/libexec/bin/shutdown.sh
  * rsync -avr —delete "PROJECT_DERECTORY/target/CONTEXT_NAME" "TOMCAT_PATH/wepapps/"
  * TOMCAT_PATH/libexec/bin/startup.sh

  

### 자동화를 위한 쉘 스크립트 작성

* deploy.sh

```shell
#!/bin/bash
cd /Users/njkim/Workspace/github/SLiPP/apps/servlet_jsp_project_SLiPP
pwd

git pull
mvn clean package

/usr/local/Cellar/tomcat/9.0.5/libexec/bin/shutdown.sh

rsync -avr —delete /Users/njkim/Workspace/github/SLiPP/apps/servlet_jsp_project_SLiPP/target/qna /usr/local/Cellar/tomcat/9.0.5/libexec/webapps

/usr/local/Cellar/tomcat/9.0.5/libexec/bin/startup.sh
```

