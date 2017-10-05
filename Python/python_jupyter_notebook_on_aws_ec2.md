# Python Jupyter Notebook on AWS EC2(ubuntu)

> 설치환경
>
> OS : Ubuntu 16.04 LTS(AWS EC2 리눅스 인스턴스)
>
> [EC2 인스턴스 생성](https://github.com/namjunemy/TIL/blob/master/AWS/aws_ec2_create_instance.md)
>
> [EC2 원격 접속](https://github.com/namjunemy/TIL/blob/master/AWS/aws_ec2_connect_and_scp.md)
>
> python을 직접 설치하거나 배포판인 Anaconda를 설치한다.

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




### python-pip 설치

pip는 파이썬으로 작성된 패키지 소프트웨어를 설치,관리하는 패키지 관리 시스템이다. Python Package Index(PyPI)에서 많은 파이썬 패키지를 찾을 수 있다. 파이썬 2.7.9 이후 버전과 파이썬 3.4 이후 버전은 pip를 기본적으로 포함한다.

pip가 설치되어 있지 않다면, 다음 명령어로 설치하고 확인 할 수 있다.

* ```$ sudo apt install python3-pip```
* ```$ pip3 -h```
* ```$ pip3 --version```




## Anaconda 설치하기

 위와 같이 파이썬을 직접 설치 할 수도 있지만, 주요 패키지가 포함된 파이썬 배포판을 사용하는 경우도 있습니다. 이런 파이썬 배포판은 데이터 처리 및 분석에 필요한 패키지가 모두 들어있어서 데이터 과학 분야에서 널리 사용되고 있습니다.

* EC2 인스턴스에 SSH를 통해 접속한다.

* 아래의 명령을 통해 installer 를 다운받는다. 버전업 되는 경우 [https://www.continuum.io/downloads](https://www.continuum.io/downloads) 이곳으로 이동하여 Installer 버튼의 오른쪽 마우스 Copy Link Address를 통해 최신버전을 다운 받을 수 있다.

  * ```$ wget https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh``` 

* 아래의 명령을 통해 아나콘다3을 설치한다.

  * ```$ bash Anaconda3-4.4.0-Linux-x86_64.sh``` 

* .bashrc 파일에 아래 내용을 추가하고 저장한다.

  * ```$ vi .bashrc``` 

  * ```shell
    # added by Anaconda3 4.4.0 installer
    export PATH="/home/ec2-user/anaconda3/bin:$PATH"
    ```

* 아나콘다3을 기본 파이썬 환경으로 설정한다.

  * ```$ which python /usr/bin/python```
  * ```source .bashrc```



## Jupyter Notebook 접근을 위한 패스워드 생성

* ipython을 통해서 Jupyter Notebook에 접근 할 비밀번호를 생성한다.

  * ```$ sudo apt insatll ipython```

  * ipython 콘솔로 진입해서 아래의 명령을 수행한다.

    * ```$ ipyhon```

    * ```In [1]: from IPython.lib import passwd```

    * ```shell
      In [2]: passwd()
      Enter password:
      Verify password:
      Out[2]: 'sha1:123154~~~~~~~~~~~'
      ```

    * Out[2]: 에 출력되는 비밀번호를 복사해 둔다. 꼭!!

    * ```In [3]: exit```




## Jupyter(Python IDE) 설치

python 개발에 많이 사용되는 IDE로

* Jupyter(iPython)
* Jetbrains PyCharm
* Sublime Text, Atom
* ...

여러가지 개발 환경이 존재한다. 이 중 Jupyter를 설치하여 사용하려고 한다. Jupyter는 iPython이라는 이름으로 제공되었던 Python 개발 도구 중에 하나로 notebook 형태로 파일 공유가 가능하고, 코더가 짠 코드를 깔끔하게 정리해서 보여준다는 장점이 있다. 게다가 웹 브라우저를 사용하고 마크다운을 지원하기 때문에 굉장히 매력적면서, 다른 언어(Ruby, R, JavaScript 등)들도 지원한다.

* 아래의 명령을 통해 jupyter를 설치한다.
  * ```$ pip3 install --upgrade jupyter```

* jupyter config 파일을 생성한다.

  * ```$ jupyter notebook --generate-config```
  * 경로는 .jupyter/jupyter_notebook_config.py 이다.

* EC2에서 jupyter notebook접근을 위해 pem파일을 업로드하고 등록한다.

  * ```$ cd```
  * ```$ mkdir certs```
  * ```$ cd certs```
  * ```$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout test.pem -out test.pem```
  * 물론 pem파일이 certs 디렉토리에 있는 상태에서 수행한다.([scp를 통한 EC2 파일 업로드 링크](https://github.com/namjunemy/TIL/blob/master/AWS/aws_ec2_connect_and_scp.md))

* jupyter notebook config파일 수정

  * ```$ vi .jupyter/jupyter_notebook_config.py```

  * 아래의 내용을 주석 해제하고 수정 또는추가한다.

  * ```shell
    c = get_config()
    ```


    shell
    # Notebook config  
    
    c.NotebookApp.certfile = u'/home/ubuntu/certs/mycert.pem' 
     
    #location of your certificate file  
    
    c.NotebookApp.ip = '*' 
     
    #so that the ipython notebook does not opens up a browser by default  
    
    c.NotebookApp.open_browser = False 
     
    #edit this with the SHA hash that you generated after typing in Step 9  
    
    c.NotebookApp.password = 'sha1:262....your hash here.........65f' 
     
    # This is the port we opened in Step 3.  
    
    c.NotebookApp.port = 8888
    

## Jupyter Notebook 접속

위의 config파일 수정으로 Jupyter Notebook의 설치와 설정은 끝났다. 작업디렉토리를 만들고 접속한다.

* ```$ mkdir Notebooks```

* ```$ cd Notebooks```

* ```$ jupyter notebook```

  * ```shell
    [I 08:58:50.434 NotebookApp] Serving notebooks from local directory: /home/ubuntu/Notebooks
    [I 08:58:50.435 NotebookApp] 0 active kernels
    [I 08:58:50.435 NotebookApp] The Jupyter Notebook is running at:
    [I 08:58:50.435 NotebookApp] https://[all ip addresses on your system]:8888/
    [I 08:58:50.435 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
    ```

  * 위와 같은 화면을 보지 못했다면, 중간과정에 오류가 있었을 것이다...

* 웹 브라우저를 열고 8888번 포트로 접속하면 경고 메세지가 뜨는데,  아래의 추가 정보에서 웹페이지로 이동 하면 된다.

  * 필수적으로 AWS EC2 Security Group의 inbound 규칙 port 8888이 open되어 있어야 한다!
  * ```https://[자신의 EC2 Public DNS]:8888```
  * 위에서 ipython 콘솔을 통해 생성했던 패스워드를 입력하면 로그인에 성공할 수 있다.

* 접속 후 Files -> New -> Python3(또는 설치된 파이썬) 선택 한 후 정상 동작 하는지 테스트 한다.

  * ```py
    x = 10;

    print(x);

    10
    ```

  * 정상적으로 Run 됐다면 .ipynb 파일이 EC2의 Notebooks 디렉토리에 생성된 것을 확인 할 수 있다.

이것으로 Jupyter Notebook 설치와 환경셋팅을 마친다.