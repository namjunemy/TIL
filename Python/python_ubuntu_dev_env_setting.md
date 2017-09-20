# 파이썬 개발 환경 설정하기

> 설치환경
>
> OS : Ubuntu 16.04 LTS(AWS EC2 리눅스 인스턴스)



##  python 설치

python은 1991년에 발표된 고급 프로그래밍 언어로 플랫폼 독립적이며 인터프리터식, 객체지향적, 동적 타이핑 대화형 언어이다. 파이썬은 비영리 파이선 소프트웨어 재단이 관리하는 개방형, 공동체 기반 개발 모델을 가지고 있다. C언어로 구현된 C파이썬 구현이 사실상의 표준이다.

python을 설치 하기전, 

* ```$ sudo apt update```
* ``` $ sudo apt upgrade```

### Ubuntu 14.04 and 16.04

* ```$ sudo add-apt-reposiroty ppa:jonathonf/python-3.6```
* ```$ sudo apt update```
* ```$ sudo apt install python3.6``` 

또는

* ```$ sudo add-apt-repository ppa:deadsnakes/ppa```

- ```$ sudo apt update```
- ```$ sudo apt install python3.6``` 

### Ubuntu 16.10 and 17.10

* ```$ sudo apt update```
* ```$ sudo apt install python3.6```

### 설치 확인

* ```$ python3.6```

  ```shell
  Python 3.6.2 (default, Jul 20 2017, 08:43:29)
  [GCC 5.4.1 20170519] on linux
  Type "help", "copyright", "credits" or "license" for more information.

  ```



## python-pip 설치

pip는 파이썬으로 작성된 패키지 소프트웨어를 설치,관리하는 패키지 관리 시스템이다. Python Package Index(PyPI)에서 많은 파이썬 패키지를 찾을 수 있다. 파이썬 2.7.9 이후 버전과 파이썬 3.4 이후 버전은 pip를 기본적으로 포함한다.

pip가 설치되어 있지 않다면, 다음 명령어로 설치하고 확인 할 수 있다.

* ```$ sudo apt install python3-pip```
* ```$ pip3 -h```
* ```$ pip3 --version```



## Jupyter(Python IDE) 설치

python 개발에 많이 사용되는 IDE로

* Jupyter(iPython)
* Jetbrains PyCharm
* Sublime Text, Atom
* ...

여러가지 개발 환경이 존재한다. 이 중 Jupyter를 설치하여 사용하려고 한다. Jupyter는 iPython이라는 이름으로 제공되었던 Python 개발 도구 중에 하나로 notebook 형태로 파일 공유가 가능하고, 코더가 짠 코드를 깔끔하게 정리해서 보여준다는 장점이 있다. 게다가 웹 브라우저를 사용하기 때문에 마크다운을 지원하기 때문에 굉장히 매력적면서, 다른 언어(Ruby, R, JavaScript 등)들도 지원한다.

설치하기 이전에 설치 위치로 

* ```$ pip3 install --upgrade jupyter```

