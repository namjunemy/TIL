# python 02

>Contents
>
>* 모듈
>  * 두 개의 소스파일로 만드는 하나의 프로그램 예제
>  * import에 대하여
>  * 모듈을 찾아서
>  * 메인 모듈과 하위 모듈
>* 패키지
>  * \__init__.py에 대하여
>  * site-package에 대하여

## 모듈

* 모듈
  * 일반적으로는 "독자적인 기능을 갖는 구성 요소"를 의미
  * **파이썬에서는 개별 소스 파일을 말한다.**
* 표준 모듈
  * 설치시 기본적으로 따라오는 모듈
* 사용자 생성 모듈
  * 직접 작성한 모듈
* 서드파티 모듈
  * 파이썬 재단, 프로그래머도 아닌 그 외의 사용자들에 의해 제공 되는 모듈

### 두 개의 소스파일로 만드는 하나의 프로그램

* Calculator.py

```python
def add(x, y):
    return x + y

def sub(x, y):
    return x - y

def mul(x, y):
    return x * y

def div(x, y):
    return x / y
```

* CalculatorEx.py

```python
import Calculator
or
#import Calculator as c
#from Calculator import add, sub, mul, div

print(Calculator.add(4, 3))
print(Calculator.sub(4, 3))
print(Calculator.mul(4, 3))
print(Calculator.div(4, 3))
```

### 모듈을 찾아서

* import문을 실행할 때 파이썬이 모듈 파일을 찾는 순서
  * 파이썬 인터프리터 내장 모듈
  * sys.path에 정의되어 있는 디렉토리
* sys.builtin_module_names를 출력하면 파이썬에 내장되어 있는 모듈의 목록을 볼 수 있음
  * import문이 실행되면 파이썬은 가장 먼저 이 목록을 확인함

### 메인 모듈과 하위 모듈

* 메인 모듈 : 최상위 수준으로 실행되는 스크립트
  * 파이썬에는 내장 전역 변수인 \_\_name\_\_이 있는데, 이 변수는 모듈이 최상위 수준으로 실행될 때 '\_\__main\_\_'으로 실행
  * 다른 모듈에서 import되서 사용 될 때는 실행되지 않음

```python
def add(x, y):
    return x + y

def sub(x, y):
    return x - y

def mul(x, y):
    return x * y

def div(x, y):
    return x / y

if __name__=='__main__':
    print(add(100,40))
```

```python
import Calculator

print(__name__)
if __name__=='__main__':
    print(Calculator.add(4, 3))
    print(Calculator.sub(4, 3))
    print(Calculator.mul(4, 3))
    print(Calculator.div(4, 3))
```

```python
__main__
7
1
12
1.3333333333333333
```

## 패키지

* 모듈을 모아놓는 디렉토리
* 디렉토리가 "파이썬의 패키지"로 인정받으려면 \_\_init\_\_.py 파일을 그 경로에 갖고 있어야 함
* 패키지 안의 모듈을 import하려면
  * `from 패키지 import 모듈`로 모듈을 불러온다.

### \_\_init\_\_.py에 대하여

* 보통의 경우, \_\_init\_\_.py 파일은 비워둠
* 이 파일을 손대는 경우는 \_\_all\_\_이라는 변수를 조정할 때 정도
  * \_\_all\_\_은 다음과 같은 코드를 실행할 때 패키지로부터 반입할 모듈의 목록을 정의하기 위해 사용
  * `form 패키지 import *`
* import * 은 사용을 자제하는 것이 좋음

### site-package에 대하여

* 각종 서드 파티 모듈을 여기에 설치함
* sys.path에 있던 stie-package 디렉토리에 새로운 디렉토리를 만들고 그 안에 \_\_init\_\_.py와 모듈을 만들고 사용

  

