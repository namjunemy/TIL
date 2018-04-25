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

  

## 객체 지향 프로그래밍과 클래스

> Contents
>
> * 객체 지향 프로그래밍
>   * 객체와 클래스
> * 클래스의 정의
>   * _\_init\_\_() 메소드를 이용한 초기화
>   * self에 대하여
>   * 정적 메소드와 클래스 메소드
>   * private 멤버
> * 상속
>   * super()
>   * 다중상속
>   * 오버라이딩
> * 데코레이터
> * for문으로 순회 할 수 있는 객체 만들기
>   * iterator
>   * generator
> * 상속의 조건 : 추상 기반 클래스

### 객체와 클래스

* 객체(Object) = 속성(Attribute) + 기능(Method)
* 속성은 사물의 특징
  * 자동차의 바퀴의 크기, 엔진의 배기량, 색
* 기능은 어떤 것의 특징적인 동작
  * 자동차의 전진, 후진, 좌회전, 우회전
* 클래스 정의
  * `class Car:`
* 클래스 안의 \_\_init\_\_() 메소드는 생성자의 역할을 한다.
* 객체와 관련된 멤버들은 항상 메소드 안에 멤버 변수 self를 가지고 있는다.

```python
class Calculator:

    def __init__(self):
        self.datas = []

    def dataAdd(self, data):
        self.datas.append(data)

    def datasAdd(self, data):
        self.datas.extend(data)

    def sum(self):
        tot = 0
        for n in self.datas:
            tot += n
        return tot

    def multiple(self):
        mul = 1
        for n in self.datas:
            mul *= n
        return mul

    
if __name__ == '__main__':
    calc = Calculator()
    calc.dataAdd(10)
    calc.datasAdd([1, 2, 3])
    print(calc.sum())
    print(calc.multiple())
```

```shell
16
60
```

```python
class Account:

    def __init__(self, id, name, balance):
        self.id = id
        self.name = name
        self.balance = balance
    
    def deposit(self, money):
        self.balance += money
    
    def withdraw(self, money):
        self.balance -= money
    
    def info(self):
        return '계좌번호: {0}, 이름: {1}, 잔액: {2}'.format(self.id, self.name, self.balance)


if __name__ == '__main__':
    acc = Account('1001', '김남준', 10000000)
    print(acc.info())
    acc.deposit(20000)
    print(acc.info())    
    acc.withdraw(5000)
    print(acc.info())
```

```python
계좌번호: 1001, 이름: 김남준, 잔액: 10000000
계좌번호: 1001, 이름: 김남준, 잔액: 10020000
계좌번호: 1001, 이름: 김남준, 잔액: 10015000
```

* 클래스를 활용해서 은행, 계좌 구현

```python
class Account:

    def __init__(self, id, name, balance):
        self.id = id
        self.name = name
        self.balance = balance
    
    def deposit(self, money):
        self.balance += money
    
    def withdraw(self, money):
        self.balance -= money
    
    def info(self):
        return '계좌번호: {0}, 이름: {1}, 잔액: {2}'.format(self.id, self.name, self.balance)


if __name__ == '__main__':
    acc = Account('1001', '김남준', 10000000)
    print(acc.info())
    acc.deposit(20000)
    print(acc.info())    
    acc.withdraw(5000)
    print(acc.info())
```

```python
from BankEx.account import Account



class Bank:

    def __init__(self):
        self.accList = {}
    
    def menu(self):
        print('NJ은행')
        print('0.종료')
        print('1. 계좌개설')
        print('2. 입금')
        print('3. 출금')
        print('4. 계좌 조회')
        print('5. 전체 계좌 조회')
        print('선택 >> ', end='')
        select = int(input())
        return select
        
    def makeAccount(self):
        print('[계좌 개설]')
        print('계좌번호: ', end='')
        accId = input()
        print('이름: ', end='')
        name = input()
        print('입금액: ', end='')
        money = int(input())
        if accId in self.accList.keys():
            print('이미 존재하는 계좌번호 입니다.')
            return
        self.accList[accId] = Account(accId, name, money)

    def deposit(self):
        print('[입금]')
        print('계좌번호 : ', end='')
        accId = input()
        print('입금액: ', end='')
        money = int(input())
        if accId not in self.accList.keys():
            print('계좌가 존재하지 않습니다.')
        else:
            self.accList[accId].deposit(money)

    def withdraw(self):
        print('[출금]')
        print('계좌번호 : ', end='')
        accId = input()
        print('입금액: ', end='')
        money = int(input())
        if accId not in self.accList.keys():
            print('계좌가 존재하지 않습니다.')
        else:
            self.accList[accId].withdraw(money)

    def accInfo(self):
        print('[계좌 조회]')
        print('계좌 번호: ', end='')
        accId = input()
        if accId not in self.accList.keys():
            print('계좌가 존재하지 않습니다.')
        else:
            print(self.accList[accId].info())

    def allAccInfo(self):
        print('[전체 계좌 조회]')
        for acc in self.accList.values():
            print(acc.info())


if __name__ == '__main__':
    bank = Bank()
    while(True):
        sel = bank.menu()
        if sel == 0: break
        elif sel == 1: bank.makeAccount()
        elif sel == 2: bank.deposit()
        elif sel == 3: bank.withdraw()
        elif sel == 4: bank.accInfo()
        elif sel == 5: bank.allAccInfo()
```

### 정적 메소드와 클래스 메소드

* 인스턴스 메소드 - 인스턴스에 속한 메소드
  * 인스턴스를 통해 호출이 가능하다는 뜻
* 정적 메소드와 클래스 메소드는 클래스에 귀속된다
* 정적 메소드
  * @staticmethod 데코레이터로 수식
  * self 키워드 없이 정의
* 클래스 메소드
  * 객체를 생성하더라도 새로 생성되지 않는 클래스 변수를 클래스 메소드를 통해서 접근 가능
  * 객체들이 클래스 변수를 공유하고 있어야 하는 상황에서 사용
  * @classmethod 데코레이터로 수식
  * cls 매개변수 사용

```python
class Card:
    width = 15
    height = 20
    def __init__(self, shape, num):
        self.shape = shape
        self.num =  num
    
    @classmethod
    def setWidth(cls, w):
        cls.width = w
        
    @classmethod
    def setHeight(cls, h):
        cls.height = h
    
    def info(self):
        print('{0},{1},{2},{3}'.format(self.shape, self.num,Card.height, Card.width))

if __name__ == '__main__':
    h1 = Card('Hart', 'A')
    dQ = Card('Dia', 'Q')
    h1.info()
    dQ.info()
    
    h1.setHeight(25)
    dQ.setWidth(20)
    h1.info()
    dQ.info()
```

```python
Hart,A,20,15
Dia,Q,20,15
Hart,A,25,20
Dia,Q,25,20
```

* 계좌의 번호 관리를 클래스 변수로 관리할 수 있다

```python
class Account:
    sid = 1000
    def __init__(self, name, balance):
        Account.sid += 1
        self.id = str(Account.sid)
        self.name = name
        self.balance = balance
    
    def deposit(self, money):
        self.balance += money
    
    def withdraw(self, money):
        self.balance -= money
    
    def info(self):
        return '계좌번호: {0}, 이름: {1}, 잔액: {2}'.format(self.id, self.name, self.balance)


if __name__ == '__main__':
    acc = Account('김남준', 10000000)
    print(acc.id)
    print(acc.info())
    acc.deposit(20000)
    print(acc.info())    
    acc.withdraw(5000)
    print(acc.info())
```

### 상속

```python
class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def info(self):
        return '이름:{0}, 나이:{1}'.format(self.name, self.age)


class Student(Person):

    def __init__(self, name, age, major):
        super().__init__(name, age)
        self.major = major

    def info(self):
        return '{0}, 전공:{1}'.format(super().info(), self.major)


if __name__ == '__main__':
    s1 = Student('namjune kim', 26, 'computer')
    print(s1.info())
```

* 상속을 활용해서 일반 계좌와 특수 계좌 구현, 고객의 등급에 따라서 입금시 이자의 개념으로 추가금 지급
* account.py

```python
class Account:
    sid = 1000
    def __init__(self, name, balance):
        Account.sid += 1
        self.id = str(Account.sid)
        self.name = name
        self.balance = balance
    
    def deposit(self, money):
        self.balance += money
    
    def withdraw(self, money):
        self.balance -= money
    
    def info(self):
        return '계좌번호: {0}, 이름: {1}, 잔액: {2}'.format(self.id, self.name, self.balance)


if __name__ == '__main__':
    acc = Account('김남준', 10000000)
    print(acc.id)
    print(acc.info())
    acc.deposit(20000)
    print(acc.info())    
    acc.withdraw(5000)
    print(acc.info())
```

* account_grade.py

```python
from bankex.account import Account


class AccountWithGrade(Account):

    def __init__(self, name, balance, grade):
        super().__init__(name, balance)
        self.grade = grade
        self.rate = 0;
        if self.grade == 'V':
            self.rate = 0.04
        elif self.grade == 'G':
            self.rate = 0.03
        elif self.grade == 'S':
            self.rate = 0.02
        elif self.grade == 'N':
            self.rate = 0.01
    
    def deposit(self, money):
        super().deposit(money + int(money * self.rate))
        
    def info(self):
        return '{0}, 등급: {1}'.format(super().info(), self.grade)


if __name__ == '__main__':
    pass
```

* bank.python

```python
from bankex.account import Account
from bankex.acccount_grade import AccountWithGrade


class Bank:

    def __init__(self):
        self.accList = {}
    
    def menu(self):
        print('NJ은행')
        print('0.종료')
        print('1. 계좌개설')
        print('2. 입금')
        print('3. 출금')
        print('4. 계좌 조회')
        print('5. 전체 계좌 조회')
        print('선택 >> ', end='')
        select = int(input())
        return select

    def accMenu(self):
        print('[계좌 개설]')
        print('1. 일반계좌')
        print('2. 특수계좌')
        print('선택 >> ', end='')
        select = int(input())
        if select == 1:
            self.makeAccount()
        elif select == 2:
            self.makeSpecialAccount()
    
    def makeAccount(self):
        print('이름: ', end='')
        name = input()
        print('입금액: ', end='')
        money = int(input())
        acc = Account(name, money)
        self.accList[acc.id] = acc
        print('계좌번호는 {0} 입니다.'.format(acc.id))
            
    def makeSpecialAccount(self):
        print('이름: ', end='')
        name = input()
        print('입금액: ', end='')
        money = int(input())
        print('등급(VIP-V,Gold-G,Silver-S,Normal-N): ', end='')
        grade = input()
        acc = AccountWithGrade(name, money, grade)
        self.accList[acc.id] = acc
        print('계좌번호는 {0} 입니다.'.format(acc.id))
        
    def deposit(self):
        print('[입금]')
        print('계좌번호 : ', end='')
        accId = input()
        print('입금액: ', end='')
        money = int(input())
        if accId not in self.accList.keys():
            print('계좌가 존재하지 않습니다.')
        else:
            self.accList[accId].deposit(money)

    def withdraw(self):
        print('[출금]')
        print('계좌번호 : ', end='')
        accId = input()
        print('입금액: ', end='')
        money = int(input())
        if accId not in self.accList.keys():
            print('계좌가 존재하지 않습니다.')
        else:
            self.accList[accId].withdraw(money)

    def accInfo(self):
        print('[계좌 조회]')
        print('계좌 번호: ', end='')
        accId = input()
        if accId not in self.accList.keys():
            print('계좌가 존재하지 않습니다.')
        else:
            print(self.accList[accId].info())

    def allAccInfo(self):
        print('[전체 계좌 조회]')
        for acc in self.accList.values():
            print(acc.info())


if __name__ == '__main__':
    bank = Bank()
    while(True):
        sel = bank.menu()
        if sel == 0: break
        elif sel == 1: bank.accMenu()
        elif sel == 2: bank.deposit()
        elif sel == 3: bank.withdraw()
        elif sel == 4: bank.accInfo()
        elif sel == 5: bank.allAccInfo()
```

```python
NJ은행
0.종료
1. 계좌개설
2. 입금
3. 출금
4. 계좌 조회
5. 전체 계좌 조회
선택 >> 1
[계좌 개설]
1. 일반계좌
2. 특수계좌
선택 >> 2
이름: knj
입금액: 0
등급(VIP-V,Gold-G,Silver-S,Normal-N): V
계좌번호는 1001 입니다.

NJ은행
0.종료
1. 계좌개설
2. 입금
3. 출금
4. 계좌 조회
5. 전체 계좌 조회
선택 >> 2
[입금]
계좌번호 : 1001
입금액: 10000

NJ은행
0.종료
1. 계좌개설
2. 입금
3. 출금
4. 계좌 조회
5. 전체 계좌 조회
선택 >> 5
[전체 계좌 조회]
계좌번호: 1001, 이름: knj, 잔액: 10400, 등급: V
```

### 다중상속

* 파이썬은 다중상속을 허용한다
* 파생 클래스의 정의에 기반 클래스의 이름을 , 로 구분해서 넣으면 다중 상속이 이루어 진다.
* `class D(A, B, C)`

### 데코레이터

* 함수를 꾸미는 객체이다.
* 데코레이터는 \_\_call\_\_() 메소드를 구현하는 클래스
  * _\_call\_\_() 메소드는 객체를 함수 호출 방식으로 사용하게 만드는 메소드
  * 인스턴스 변수에 ()를 붙여서 사용

```python
class Temp:
    def __call__(self):
        print('함수가 호출됨')

if __name__ == '__main__':
    f = Temp()
    f()
```

```pyton
함수가 호출됨
```

```python
class Temp:

    def __init__(self, f):
        print("Initializing MyDecorator...")
        self.func = f

    def __call__(self):
        self.func()
        print('함수가 호출됨')


@Temp
def myfunc():
    print('내가 만든 함수')


if __name__ == '__main__':
    myfunc()
```

### 이터레이터

* 순회 하려는 객체의 \_\_iter\_\_() 메소드를 호출하는 것

```python
iter = range(3).__iter__()
print(iter.__next__())
print(iter.__next__())
print(iter.__next__())
```

### 추상 클래스

* 자식 클래스가 갖춰야 할 특징을 강제
* 추상 클래스를 상속받는 클래스는 abstract 메소드를 반드시 오버라이딩 해야 객체 생성이 가능하다.

