# python 03

> 뇌를 자극하는 파이썬 3 - 박상현

## 오류를 어떻게 다뤄야 할까

> Contents
>
> * 예외란
> * try ~ except로 예외 처리하기
>   * 복수 개의 except절 사용
>   * try절을 무사히 실행하면 만날 수 있는 else
>   * 어떤 일이 있어도 반드시 실행되는 finally
> * Exception 클래스
> * 우리도 예외 좀 일으켜 보자
> * 내가 만든 예외 형식

### 예외란

* 파이썬에서 Exception은 문법적으로는 문제가 없는 코드를 실행하는 중에 발생하는 오류
  * ValueError

```python
def my_power(y):
    print('숫자를 입력하세요')
    x = input()
    return int(x) ** y


if __name__ == '__main__':
    print(my_power(2))
```

```python
숫자를 입력하세요
a
Traceback (most recent call last):
  File "C:\NJ\workspace\0426\exception\value_error.py", line 8, in <module>
    print(my_power(2))
  File "C:\NJ\workspace\0426\exception\value_error.py", line 4, in my_power
    return int(x) ** y
ValueError: invalid literal for int() with base 10: 'a'
```

### try ~ except로 예외 처리하기

* try 절 안에는 문제가 없을 경우의 코드 블록을 배치
* except 절에는 문제가 생겼을 때 처리하는 코드 블록을 배치

```python
try:
    # 문제가 없을 경우 코드 블록
except:
    # 문제가 생겼을 때 실행할 코드
```

* ex

```python
try:
    s1 = input()    
    s2 = input()
    print(int(s1) + int(s2))
except:
        print('숫자를 입력하세요')
```

* 복수의 except절로 처리

```python
try:
    s1 = input()    
    s2 = input()
    print(int(s1) / int(s2))
except ValueError:
    print('숫자를 입력하세요')
except ZeroDivisionError:
    print('0으로 나눌 수 없습니다')
```

* 반복문에서 try ~ except 처리

```python
ar1 = [10, 20, 30]
ar2 = [5, 0]

for i in range(len(ar1)):
    try: 
        print(ar1[i] / ar2[i])
    except ZeroDivisionError:
        
        
        print('0으로 나눌 수 없습니다.')
    except:
        print('나누는 데이터가 부족합니다.')
```

```python
ar1 = [10, 20, 30]
ar2 = [5, 0]

for i in range(len(ar1)):
    try: 
        print(ar1[i] / ar2[i])
    except ZeroDivisionError as err:
        print(err)
    except IndexError as err:
        print(err)
```

```shell
2.0
division by zero
list index out of range
```

* try절을 무사히 실행하면 만날 수 있는 else

```python
try:
    #
except:
    #
else:
    # except절을 만나지 않았을 경우 실행하는 코드 블록
```

* 어떤 일이 있어도 반드시 싱행되는 finally
  * finally는 무조건 실행
  * 파일이나 통신 채널과 같은 자원을 정리할 때 사용됨

### 우리도 예외 좀 일으켜 보자

* raise 문을 사용하면 직접 예외를 발생시킬 수 있음
* Exception 객체를 생성

```python
def deposit(money):
    if money <= 0:
        try:
            raise Exception('입금액은 1원 이상이어야 합니다.')
        except Exception as err:
            print(err)
    
deposit(-1000)
```

```python
def deposit(money):
    if money <= 0:
            raise Exception('입금액은 1원 이상이어야 합니다.')

try:
    deposit(-1000)
except Exception as err:
    print(err)
```

### 내가 만든 예외 형식

* 파이썬이 제공하는 내장 예외 형식만으로 충분하지 않을 때 직접 예외 클래스를 정의할 수 있음
* 사용자 정의 예외 클래스는 Exception 클래스를 상속하여 정의함

```python
class MyException(Exception):
    pass
```

* 필요에 따라 데이터 속성이나 메소드를 추가 할 수 있다

```python
class MyException(Exception):
    def __init__(self):
        super().__init__( "MyException이 발생했습니다." )
```

* bank 프로젝트에 예외 클래스 적용

```python
class ERRCODE:
    EXIST_ACC = 100
    NOTEXIST_ACC = 101
    DEPOSIT = 102
    WITHDRAW = 103
    MENU = 104
    ACCMENU = 105

    def __setattr__(self, *_):
        pass


class AccountException(Exception):

    def __init__(self, code):
        super().__init__('계좌오류')
        self.code = code

    def __str__(self, *args, **kwargs):
        msg = super().__str__()
        if self.code == ERRCODE.EXIST_ACC:
            msg += ":" + '이미 존재하는 계좌번호'
        elif self.code == ERRCODE.NOTEXIST_ACC:
            msg += ":" + '존재하지 않는 계좌'
        elif self.code == ERRCODE.DEPOSIT:
            msg += ":" + '입금액이 잘못되었습니다'
        elif self.code == ERRCODE.WITHDRAW:
            msg += ":" + '잔액이 부족합니다'
        elif self.code == ERRCODE.MENU:
            msg += ":" + '0~5 사이의 메뉴를 선택해주세요'
        elif self.code == ERRCODE.ACCMENU:
            msg += ":" + '1,2번만 선택이 가능합니다'
        return msg
```

```python
from exception.account_exception import AccountException, ERRCODE


class Account:
    sid = 1000
    def __init__(self, name, balance):
        Account.sid += 1
        self.id = str(Account.sid)
        self.name = name
        self.balance = balance
    
    def deposit(self, money):
        if money <= 0:
            raise AccountException(ERRCODE.DEPOSIT)
        self.balance += money
    
    def withdraw(self, money):
        if money > self.balance:
            raise AccountException(ERRCODE.WITHDRAW)
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

```python
from acc.acccount_grade import AccountWithGrade
from acc.account import Account
from exception.account_exception import AccountException, ERRCODE


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
        if select < 0 or select > 5:
            raise AccountException(ERRCODE.MENU)
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
        else:
            raise AccountException(ERRCODE.ACCMENU)
    
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
            raise AccountException(ERRCODE.NOTEXIST_ACC)
        self.accList[accId].deposit(money)

    def withdraw(self):
        print('[출금]')
        print('계좌번호 : ', end='')
        accId = input()
        print('입금액: ', end='')
        money = int(input())
        if accId not in self.accList.keys():
            raise AccountException(ERRCODE.NOTEXIST_ACC)
        self.accList[accId].withdraw(money)

    def accInfo(self):
        print('[계좌 조회]')
        print('계좌 번호: ', end='')
        accId = input()
        if accId not in self.accList.keys():
            raise AccountException(ERRCODE.NOTEXIST_ACC)
        print(self.accList[accId].info())

    def allAccInfo(self):
        print('[전체 계좌 조회]')
        for acc in self.accList.values():
            print(acc.info())


if __name__ == '__main__':
    bank = Bank()
    while(True):
        try:
            sel = bank.menu()
            if sel == 0: break
            elif sel == 1: bank.accMenu()
            elif sel == 2: bank.deposit()
            elif sel == 3: bank.withdraw()
            elif sel == 4: bank.accInfo()
            elif sel == 5: bank.allAccInfo()
        except AccountException as err:
            print(err)
```

