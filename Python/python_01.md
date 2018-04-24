# python 01

> Contents
>
> - 파이썬 설치하기
> - IDLE의 두 가지 모드
>   - 파이썬 shell로 코딩
>   - 코드 편집기로 코딩
> - 주석

## 01. 설치하기

- https://www.python.org/downloads/
- python3.6의 windows installer에는 PATH를 추가해주는 메뉴가 추가됨
  - 별도의 환경 변수 설정 필요 없음

## 02. eclipse - python 개발

- eclipse -> help -> eclipse marketplace -> pydev -> python ide for Eclipse 6.3.2 설치 후 개발
- 설치 후 new project에서 python 프로젝트(PyDev Project) 생성 가능
- windows -> preference -> PyDev -> Interpreters -> Python Interpreter -> Quick Auto-Config - 설치한 라이브러리 연동
- new Project -> PyDev Project -> python module로 python 코드 작성

## 03. 데이터 다루기 - 수와 텍스트, 그리고 비트

> * 변수
> * 수 다루기
>   * 정수
>   * 실수
>   * 복소수
>   * Math 모듈을 이용한 계산
> * 텍스트 다루기
>   * 문자열 메소드
> * 수와 텍스트 변환
> * 비트 다루기
>   * 시프트 연산자
>   * 비트 논리 연산자

### 변수

* 변수는 데이터를 담는 메모리 공간
* 변수에는 수, 텍스트, 목록, 이미지 데이터 등을 담을 수 있음

### 수 다루기

* 텍스트, 이미지, 음성 등 컴퓨터의 다양한 데이터 처리는 수 처리를 응용하여 이루어짐
* 파이썬이 기본적으로 지원하는 세 종류의 수
  *  정수
    * 숫자의 크기 제한이 없이 저장 가능
  * 실수 - 8byte double
  * 복소수
* 숫자의 type확인

```python
>>> b = 3.5
>>> type(b)
<class 'float'>
```

#### 정수

* 음의 정수, 0, 양의 정수
* 파이썬에서는 메모리가 허용하는 한, 무한대의 정수를 다룰 수 있음
* 파이썬에서 제공하는 사칙 연산자
  * +, -, *, /, %
  * 나눗셈의 몫 구하기 : //
  * `m = 10 // 3`
  * m = 3
* 16진수 접두사 : 0x
* 2진수 접두사 : 0b
* 8진수 접두사 : 0o
* 10진수를 16진수 문자열로 변환
  * hex()
* 16진수 데이터를 변수에 할당
  * a = 0xFF
  * 255
* 10진수를 2진수 문자열로 변환
  * bin()
* 10진수를 8진수 문자열로 변환
  * oct()

#### 실수

* 파이썬에서는 실수를 지원하기 위해 부동 소수형을 제공

  * 부동 소수형은 소수점을 움직여서 소스를 표현하는 자료형

* 부동 소수형의 특징

  * 부동 소수형은 8바이트만을 이용해서 수를 표현한다. 즉, 한정된 범위의 수만 표현할 수 있다.
  * 디지털 방식으로 소수를 표현해야 하기 때문에 정밀도의 한계를 갖는다.

* ex

  * 22 / 7 의 결과는 무리수지만 부동소수형은 소수점 이하 15자리만 표현

  ```python
  >>> b = 22 / 7
  >>> b
  3.142857142857143
  >>> type(b)
  <class 'float'>
  ```

#### 복소수

* 복소스는 a +_ bi의 꼴로 나타낼 수 있는 수

  * 파이썬에서는 허수 단위를 나타내는 부호로 i 대신 j를 사용

  ```python
  >>> com = 3 + 5j
  >>> com
  (3+5j)
  >>> com.real
  3.0
  >>> com.imag
  5.0
  >>> com.conjugate()
  (3-5j)
  ```

### Math 모듈을 이용한 계산

* math 모듈을 사용하기 위한 import문
* python에서 모듈은 소스 파일

```python
>>> import math
>>> math.pi
3.141592653589793
>>> math.e
2.718281828459045
```

* 절대값 : abs()
* 반올림 : round()
* 버림 : math 모듈 내의 trunc()

```python
>>> abs(-100)
100
>>> round(15.6)
16
>>> math.trunc(15.6)
15
```

* 팩토리얼

```python
>>> math.factorial(5)
120
```

* 제곱과 제곱근

```python
>>> 3 ** 3
27
>>> math.pow(3,3)
27.0
>>> math.sqrt(81)
9.0
>>> 27 **  (1/3)
3.0
>>> math.pow(81,0.5)
9.0
```

* 로그
  	* 첫 번째 매개변수의 로그를 구한다. 두 번째 매개변수는 밑수, 생략하면 e

```python
>>> math.log(4,2)
2.0
>>> math.log(math.e)
1.0
```

### 텍스트 다루기

* 프로그래밍 언어마다 방식이 조금씩 다르긴 하지만 대부분은 개별 문자를 나타내는 수를 이어서 텍스트를 표현
* 파이썬에서는 텍스트를 다루는 자료형으로 string을 제공
  * string의 뜻은 끈, 줄 등의 뜻을 가지고 있음
  * 우리말로는 문자열, 문자열도 문자를 가지런히 늘어놨다는 뜻
  * 문자열은 항상 끝에 끝을 알리는 null이 붙어 있다.

```python
>>> s = 'hello'
>>> s
'hello'
>>> s = "hello"
>>> s
'hello'
>>> "'it'"
"'it'"
```

```python
>>> str = '''hello
python,
nice to meet you'''
>>> str
'hello\npython,\nnice to meet you'
```

* 여러 라인을 주석으로 하고 싶을 때도 '''사용

```python
'''
여러 라인의
주석의
내용
'''
```

* 문자열을 다룰 때의 + 연산자는 두 문자열을 하나로 이어 붙임
* 문자열 분리(slicing) 연산자는 []을 사용함

```python
>>> s1 = 'hello'
>>> s2 = ' python'
>>> s1 + s2
'hello python'
>>> s3 = s1 + s2
>>> s3[0]
'h'
>>> s3[0:5]
'hello'
>>> s3[5:]
' python'
```

* in 연산자는 원하는 문자열이 특정 문자열 안에 존재하는지를 확인

```python
>>> 'python' in s3
True
```

* len()은 문자열의 길이를 구하는 함수

```python
>>> len(s3)
12
```

* startwith()
* endswith()
* find()

```python
>>> str = 'hello world'
>>> str.startswith('h')
True
>>> str.endswith('w')
False
>>> str.find('hello')
0
```

- rfind()
  - 뒤에서부터 find
- count()
  - 특정 문자열의 포함 여부를 카운트
- lstrip()
  - 원본 문자열 왼쪽에 있는 공백을 제거
- rstrip()
  - 원본 문자열 오른쪽에 있는 공백을 제거
- strip()
  - 문자열 처음과 끝의 공백을 제거
- String은 immutable객체 이므로 문자열 연산을 하면 새로운 string 객체가 생성된다.
- isalpha()
  - 문자열이 알파벳으로만 이루어져 있는지 평가
- isnumeric()
  - 원본 문자열이 수로만 이루어져 있는지 평가
- isalnum()
  - 원본 문자열이 알파벳과 수로만 이루어져 있는지 평가
- replace()
  - 원본 문자열에서 찾고자 하는 문자열을 바꾸고자 하는 문자열로 변경
  - 마찬가지로 immutable 특성을 가지므로 새로운 객체 생성
- split()
  - 매개변수로 입력한 문자열을 기준으로 원본 문자열을 나누어 리스트를 만든다.
- upper()
  - 모두 대문자로 변경
- lower()
  - 모두 소문자로 변경
- format()
  - 문자열의 format을 지정해서 사용
  - string의 메소드

```python
>>> name = "namjune"
>>> age = 20
>>> '이름:{0}, 나이:{1}'.format(name, age)
'이름:namjune, 나이:20'
```

### 수에서 텍스트로 텍스트에서 수로

* input()
  * 사용자로 부터 입력을 받는 input()은 문자열을 반환한다.
* input()을 통해 입력받은 문자열을 숫자로 casting해서 연산할 수 있다.
  * int(), float(), complex()

```python
>>> a = input()
100
>>> b = input()
200
>>> result = a + b
>>> result
'100200'
>>> result = int(a) + int(b)
>>> result
300
```

```python
>>> name = input()
Namjun Kim
>>> age = int(input())
26
>>> height = float(input())
173.5
```

* 숫자를 문자열로 바꾸기 위해서는 str()을 사용
  * 문자열 연산을 위해 자주 사용함

## 04. 데이터 다루기 : List와 Tuple, Dictionary

### List

* List는 데이터의 목록을 다루는 자료형
* Slot : 리스트의 데이터를 삽입할 자리
* Element : 리스트의 요소

```python
>>> list = ['김남준', '홍길동', '양의지']
>>> list
['김남준', '홍길동', '양의지']
>>> list[2]
'양의지'
```

```python
>>> intList = [100,200,300,400,500]
>>> intList[1:4]
[200, 300, 400]
>>> list + intList
['김남준', '홍길동', '양의지', 100, 200, 300, 400, 500]
```

* 하나의 리스트가 여러가지 타입의 element를 저장하는 것이 가능하다.

#### List 메소드

* append()
* extend()
* insert()
* remove()
  * 특정 데이터 삭제
* pop()
  * 마지막 데이터 삭제 후 리턴
* index()
  * 특정 데이터의 위치
* count()
* sort()
* reverse()

```python
>>> list = [1, 2, 3]
>>> list.append(6)
>>> list
[1, 2, 3, 6]

>>> list.extend([8,9,10])
>>> list
[1, 2, 3, 6, 8, 9, 10]

>>> list.insert(0,11)
>>> list
[11, 1, 2, 3, 6, 8, 9, 10]

>>> list.remove(10)
>>> list
[11, 1, 2, 3, 6, 8, 9]

>>> list.pop()
9
>>> list
[11, 1, 2, 3, 6, 8]

>>> list.index(11)
0

>>> list.append(1)
>>> list.count(1)
2

>>> list.sort()
>>> list
[1, 1, 2, 3, 6, 8, 11]

>>> list.sort(reverse=True)
>>> list
[11, 8, 6, 3, 2, 1, 1]

>>> list.reverse()
>>> list
[1, 1, 2, 3, 6, 8, 11]
```

* 리스트를 리스트로 만들어서 사용 가능

```python
>>> a1 = [1, 2, 3]
>>> b1 = [10, 20, 30]
>>> ab = [a1, b1]
>>> ab
[[1, 2, 3], [10, 20, 30]]
>>> ab[0][1]
2
```

### Tuple

* 사전적 의미는 Tuple과 List가 비슷
  * 리스트는 목록
  * 튜플은 "N개의 요소로 된 집합"
* 파이썬의 List와 Tuple의 차이
  * List는 변경 가능(List 생성 후 추가/수정/삭제 가능)
  * Tuple은 데이터 변경 불가능(Tuple 생성 후 추가/수정/삭제 불가능)
  * List는 이름 그대로 목록 형식의 데이터를 다루는 데 적합
  * Tuple은 위경도 좌표나 RGB 색상처럼 불변의 작은 규모의 자료구조를 구성하기에 적합
* Tuple은 대괄호과 아닌 소괄호를 사용한다
  * 중괄호는 생략 가능하지만,
  * 데이터가 하나 들어가는 Tuple이라도 , 를 생략하면 안된다.

```python
>>> tuple = (1, 2, 3)
>>> tuple
(1, 2, 3)
>>> type(tuple)
<class 'tuple'>

>>> tuple2 = 10, 20, 30;
>>> type(tuple2)
<class 'tuple'>

>>> tuple3 = (10)
>>> type(tuple3)
<class 'int'>
```

* 기본적인 리스트의 메소드 사용 가능
* packing과 unpacking
  * packing : 여러 데이터를 tuple로 묶는 것
  * unpacking : tuple의 각 요소를 여러개의 변수에 할당하는 것

```python
# 패킹
>>> a = 1, 2, 3
>>> a
(1, 2, 3)

# 언패킹
>>> tu = ('hong', 20, 175.3)
>>> name, age, height = tu
>>> name
'hong'
>>> age
20
>>> height
175.3
```

```python
>>> tu = 100, 200, 300, 100, 200, 100
>>> tu.count(100)
3
>>> tu.index(200)
1
```

### Dictionary

* 사용법 측면으로 보면 리스트와 비슷
  * 리스트처럼 첨자를 이용해서 요소에 접근
* 리스트는 요소에 접근할 때 0부터 시작하는 수 첨자만 사용할 수 있지만 dictionary는 문자열과 숫자를 비롯해서 변경이 불가능한 형식이면 어떤 자료형이든 사용
  * dictionary의 첨자는 키(Key)
  * 이 키가 가리키는 슬롯에 저정되는 데이터를 Value
  * dictionary는 Key-Value Pair로 구성
* 탐색 속도가 빠르고, 사용하기도 편리
* dictionary를 만들 때는 중괄호 { } 를 이용
* 특정 슬롯에 새로운 키-값을 입력하거나 dictionary 안에 있는 요소를 참조할 때는 리스트와 튜플에서처럼 대괄호 [ ]를 이용

```python
>>> dic = {}
>>> dic['파이썬'] = 'www.python.org'
>>> dic['java'] = 'www.oracle.com/java'
>>> dic['파이썬']
'www.python.org'
>>> type(dic)
<class 'dict'>
>>> dic
{'파이썬': 'www.python.org', 'java': 'www.oracle.com/java'}
```

#### Dictionary 메소드

```python
>>> dic.keys()
dict_keys(['파이썬', 'java'])

>>> dic.items()
dict_items([('파이썬', 'www.python.org'), ('java', 'www.oracle.com/java')])

>>> dic.pop('파이썬')
'www.python.org'
>>> dic
{'java': 'www.oracle.com/java'}

>>> dic.clear()
>>> dic
{}
```

