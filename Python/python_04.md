# python 04

> 뇌를 자극하는 파이썬 3 - 박상현

## SQLite를 이용한 데이터베이스

### SQLite의 파이썬 API

* 파이썬 3에는 SQLite 라이브러라가 기본적으로 탑재되어 있다
  * `import sqlite3`
* SQLite API 사용과정
  * 커넥션(Connection) 열기
  * 커서(Cursor) 열기
  * 커서를 이용하여 데이터 추가/수정/삭제
  * 커서 닫기
  * 커넥션 닫기
* 커넥션 열고 닫기

```python
import sqlite3

con = sqlite3.connect('test.db')

con.close()
```

* 커서로 작업하기

```python
import sqlite3

con = sqlite3.connect('test.db')
cursor = con.cursor()

cursor.close()
con.close()
```

* 테이블 생성

```python
import sqlite3

con = sqlite3.connect('test.db')
cursor = con.cursor()

cursor.execute("""
CREATE TABLE PHONEBOOK
(NAME CHAR(32), PHONE CHAR(32), EMAIL CHAR(64) PRIMARY KEY)
""")

cursor.close()
con.close()
```

* 레코드 추가

```python
import sqlite3

con = sqlite3.connect('test.db')
cursor = con.cursor()

cursor.execute("""
INSERT INTO PHONEBOOK (NAME, PHONE, EMAIL)
VALUES(?, ?, ?)
""", ('박신혜', '010-1111-2222', 'psh@gmail.com'))

id = cursor.lastrowid
print(id)

cursor.execute("""
INSERT INTO PHONEBOOK (NAME, PHONE, EMAIL)
VALUES(?, ?, ?)
""", ('박신혜2', '010-1111-2222', 'psh2@gmail.com'))

id = cursor.lastrowid
print(id)

con.commit()

cursor.close()
con.close()
```

* 레코드 조회

```python
import sqlite3

con = sqlite3.connect('test.db')
cursor = con.cursor()

cursor.execute("""select name, phone, email from phonebook""")
rows = cursor.fetchall()
for row in rows:
    print('{0}, {1}, {2}'.format(row[0], row[1], row[2]))

cursor.close()
con.close()
```

* 레코드 수정

```python
import sqlite3

con = sqlite3.connect('test.db')
cursor = con.cursor()

cursor.execute("""
update phonebook set phone = ?, email = ? where name = ?
""", ('010-3333-3333', 'test@gmail.com', '박신혜'))

con.commit()

cursor.close()
con.close()
```

* 레코드 삭제

```python
import sqlite3

con = sqlite3.connect('test.db')
cursor = con.cursor()

cursor.execute("""
delete from phonebook where name = ?
""", ('박신혜2',))

con.commit()

cursor.close()
con.close()
```

### Bank 프로젝트 DB로 변경

* account_dao.py

```python
import sqlite3

from acc.acccount_grade import AccountWithGrade
from acc.account import Account


class AccountDao:

    def __init__(self):
        self.conn = sqlite3.connect('bank.db')
        
    def insert(self, acc):
        if type(acc) == AccountWithGrade:
            grade = acc.grade
        else:
            grade = ''
        cursor = self.conn.cursor()
        cursor.execute("""
        insert into account (id, name, balance, grade)
        values(?, ?, ?, ?)
        """, (acc.id, acc.name, acc.balance, grade))
        self.conn.commit()
        cursor.close()
        
    def update(self, acc):
        cursor = self.conn.cursor()
        cursor.execute("""
        update account set balance = ? where id = ?
        """, (acc.balance, acc.id))
        self.conn.commit()
        cursor.close()
    
    def select(self, aid):
        cursor = self.conn.cursor()
        cursor.execute("""
        select id, name, balance, grade from account
        where id = ?
        """, (aid,))
        rows = cursor.fetchall()
        acc = None
        for row in rows:
            if row[3] == '':
                acc = Account(row[0], row[1], row[2])
            else:
                acc = AccountWithGrade(row[0], row[1], row[2], row[3])
        cursor.close()
        return acc

    def selectAll(self):
        accs = []
        cursor = self.conn.cursor()
        cursor.execute("""
        select id, name, balance, grade from account
        """)
        rows = cursor.fetchall()
        for row in rows:
            if row[3] == '':
                accs.append(Account(row[0], row[1], row[2]))
            else:
                accs.append(AccountWithGrade(row[0], row[1], row[2], row[3]))
        cursor.close()
        return accs
    
    def destroy(self):
        self.conn.close()
```

* bank.py

```python
from acc.acccount_grade import AccountWithGrade
from acc.account import Account
from db.account_dao import AccountDao
from exception.account_exception import AccountException, ERRCODE


class Bank:

    def __init__(self):
        self.db = AccountDao()
        
    def destroy(self):
        self.db.destroy()
    
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
        acc = Account(0, name, money)
        self.db.insert(acc)
        print('계좌번호는 {0} 입니다.'.format(acc.id))
            
    def makeSpecialAccount(self):
        print('이름: ', end='')
        name = input()
        print('입금액: ', end='')
        money = int(input())
        print('등급(VIP-V,Gold-G,Silver-S,Normal-N): ', end='')
        grade = input()[0]
        acc = AccountWithGrade(0, name, money, grade)
        self.db.insert(acc)
        print('계좌번호는 {0} 입니다.'.format(acc.id))
        
    def deposit(self):
        print('[입금]')
        print('계좌번호 : ', end='')
        accId = input()
        print('입금액: ', end='')
        money = int(input())
        
        acc = self.db.select(accId)
        if not acc:
            raise AccountException(ERRCODE.NOTEXIST_ACC)
        acc.deposit(money)
        self.db.update(acc)

    def withdraw(self):
        print('[출금]')
        print('계좌번호 : ', end='')
        accId = input()
        print('입금액: ', end='')
        money = int(input())
        
        acc = self.db.select(accId)
        if not acc:
            raise AccountException(ERRCODE.NOTEXIST_ACC)
        acc.withdraw(money)
        self.db.update(acc)

    def accInfo(self):
        print('[계좌 조회]')
        print('계좌 번호: ', end='')
        accId = input()
        
        acc = self.db.select(accId)
        if not acc:
            raise AccountException(ERRCODE.NOTEXIST_ACC)
        print(acc.info())

    def allAccInfo(self):
        print('[전체 계좌 조회]')
        accs = self.db.selectAll()
        for acc in accs:
            print(acc.info())


if __name__ == '__main__':
    bank = Bank()
    while(True):
        try:
            sel = bank.menu()
            if sel == 0:
                bank.destroy()
                break
            elif sel == 1: bank.accMenu()
            elif sel == 2: bank.deposit()
            elif sel == 3: bank.withdraw()
            elif sel == 4: bank.accInfo()
            elif sel == 5: bank.allAccInfo()
        except AccountException as err:
            print(err)
```

* account_exception.py, account.py, account_grade.py는 동일

## C/C++ 연동

### C/C++ 에서 python embedding 하기

* 파이썬은 많은 기능을 가진 모듈들을 간단한 형태로 모듈화하여 제공한다. embedding하는 이유는 c++에서 복잡한 것을 파이썬으로 간단하게 해결할 수 있기 때문이다.

```c++
#include <Python.h>


int main()
{
	char filename[] = "python.py";
	FILE* fp;
	Py_Initialize();
	fp = _Py_fopen(filename, "r");
	PyRun_SimpleFile(fp, filename);
	Py_Finalize();
	
    return 0;
}
```

## 엑셀파일 다루기

### Anaconda 설치

* Anaconda 는 Continuum Analytics라는 곳에서 만든 파이썬 배포판으로 445개 정도의 파이썬 패키지를 포함하고 있다.
* https://repo.continuum.io/archive/
  * OS에 맞는 설치 파일 다운로드

### 엑셀에 데이터 저장

```python
[1] import win32com

[2] client excel =win32com.client.Dispatch("Excel.Application") 

[3] excel.Visible = True 

[4] wb = excel.Workbooks.Add() 

[5] ws = wb.Worksheets("Sheet1") 

[6] ws.Cells(1, 1).Value ="hello world" 

[7] wb.SaveAs(‘d:\hyj\work\test.xlsx') 

[8] excel.Quit()
```

### 엑셀 파일 읽기 

```python
[1] import win32com.client 

[2] excel = win32com.client.Dispatch("Excel.Application") 

[3] excel.Visible = True 

[4] wb = excel.Workbooks.Open(‘d:\\hyj\\work\\test.xlsx') 

[5] ws = wb.ActiveSheet 

[6] print(ws.Cells(1,1).Value) 

[7] excel.Quit()
```

### 셀에 컬러 입히기

* 참고로 ColorIndex 속성은 0~56까지의 값을 가질 수 있으며 숫자별로 색상이 정해져 있는데 색상 값은마이크로소프트사에서 제공. excel.application colorindex 로 검색

```python
[1] import win32com.client 

[2] excel = win32com.client.Dispatch("Excel.Application") 

[3] excel.Visible = True 

[4] wb = excel.Workbooks.Add() 

[5] ws = wb.Worksheets("Sheet1") 

[6] ws.Cells(1, 1).Value = “python"

[7] ws.Range(“B1:C1”).Value = “is good"

[8] ws.Range("A1:B1").Interior.ColorIndex=10
```

### csv 파일 2차원 배열로 읽어오기

* numpy 모듈을 사용하면 된다.

```python
import numpy

data = numpy.loadtxt(fname = 'C:\\NJ\\download\\inflammation.csv', delimiter=',')
print(data)
```

```python
[[  0.   0.   1.   3.   1.   2.   4.   7.   8.   3.   3.   3.  10.   5.
    7.   4.   7.   7.  12.  18.   6.  13.  11.  11.   7.   7.   4.   6.
    8.   8.   4.   4.   5.   7.   3.   4.   2.   3.   0.   0.]
 [  0.   1.   2.   1.   2.   1.   3.   2.   2.   6.  10.  11.   5.   9.
    4.   4.   7.  16.   8.   6.  18.   4.  12.   5.  12.   7.  11.   5.
   11.   3.   3.   5.   4.   4.   5.   5.   1.   1.   0.   1.]
 [  0.   1.   1.   3.   3.   2.   6.   2.   5.   9.   5.   7.   4.   5.
    4.  15.   5.  11.   9.  10.  19.  14.  12.  17.   7.  12.  11.   7.
    4.   2.  10.   5.   4.   2.   2.   3.   2.   2.   1.   1.]
 [  0.   0.   2.   0.   4.   2.   2.   1.   6.   7.  10.   7.   9.  13.
    8.   8.  15.  10.  10.   7.  17.   4.   4.   7.   6.  15.   6.   4.
    9.  11.   3.   5.   6.   3.   3.   4.   2.   3.   2.   1.]
 [  0.   1.   1.   3.   3.   1.   3.   5.   2.   4.   4.   7.   6.   5.
    3.  10.   8.  10.   6.  17.   9.  14.   9.   7.  13.   9.  12.   6.
    7.   7.   9.   6.   3.   2.   2.   4.   2.   0.   1.   1.]]
```

* numpy 메소드

  * data.shape : 행과 열의 수를 반환

  ```python
  print(data.shape)

  >>
  (5, 40)
  ```

  * 부분 추출

  ```python
  small = data[0:4, 0:10]
  print(small)

  >>
  [[ 0.  0.  1.  3.  1.  2.  4.  7.  8.  3.]
   [ 0.  1.  2.  1.  2.  1.  3.  2.  2.  6.]
   [ 0.  1.  1.  3.  3.  2.  6.  2.  5.  9.]
   [ 0.  0.  2.  0.  4.  2.  2.  1.  6.  7.]]
  ```

  * 모든 데이터 * 2

  ```python
  double = small * 2
  print(double)

  >>
  [[  0.   0.   2.   6.   2.   4.   8.  14.  16.   6.]
   [  0.   2.   4.   2.   4.   2.   6.   4.   4.  12.]
   [  0.   2.   2.   6.   6.   4.  12.   4.  10.  18.]
   [  0.   0.   4.   0.   8.   4.   4.   2.  12.  14.]]
  ```

  * numpy 데이터 끼리의 합

  ```python
  sum = small + double
  print(sum)

  >>
  [[  0.   0.   3.   9.   3.   6.  12.  21.  24.   9.]
   [  0.   3.   6.   3.   6.   3.   9.   6.   6.  18.]
   [  0.   3.   3.   9.   9.   6.  18.   6.  15.  27.]
   [  0.   0.   6.   0.  12.   6.   6.   3.  18.  21.]]
  ```

  * numpy 함수

  ```python
  print(data.min())
  print(data.max())
  print(data.mean())
  print(data.std())
  0.0
  19.0
  5.685
  4.27969333013

  print(data[2,:].max())
  19.0

  print(data.mean(axis=0))
  [  0.    0.6   1.4   2.    2.6   1.6   3.6   3.4   4.6   5.8   6.4   7.
     6.8   7.4   5.2   8.2   8.4  10.8   9.   11.6  13.8   9.8   9.6   9.4
     9.   10.    8.8   5.6   7.8   6.2   5.8   5.    4.4   3.6   3.    4.
     1.8   1.8   0.8   0.8]
  ```

* matplotlib으로 그래프 그리기

```python
from matplotlib import pyplot as plt

plt.figure(figsize=(10.0, 3.0))
plt.subplot(1, 3, 1)
plt.ylabel('average')
plt.plot(data.mean(0))

plt.subplot(1, 3, 2)
plt.ylabel('max')
plt.plot(data.max(0))

plt.subplot(1, 3, 3)
plt.ylabel('min')
plt.plot(data.min(0))

plt.tight_layout()
plt.show()
```

![](https://github.com/namjunemy/TIL/blob/master/Python/image/matplotlib_graph.PNG?raw=true)

## Pandas를 이용한 데이터 분석

> Contents
>
> * Pandas Series
> * Pandas DataFrame
> * DataReader
> * 주식 차트 그리기

### Pandas

* Pandas는 효가적인 데이터 분석을 위한 고수준의 자료구조와 데이터 분석 도구를 제공
* Pandas의 Series는 1차원 데이터를 다루는 데 효과적인 자료구조이며, DataFrame은 행과 열로 구성된 2차원 데이터를 다루는 데 효과적인 자료구조이다.

### Pandas Series : 1차원 자료구조

* 생성자로 파이썬 리스트를 넘겨주면 해당 값에 포함하는 Series 객체를 생성해준다.

```python
from pandas import Series, DataFrame
kakao = Series([92600, 92400, 92100, 94300, 92300]) 
print(kakao)

>>
0    92600
1    92400
2    92100
3    94300
4    92300
dtype: int64
```

```python
from pandas import Series, DataFrame
kakao = Series([92600, 92400, 92100, 94300, 92300], index=['2016-02-19', '2016-02-18', '2016-02-17', '2016-02-	16', '2016-02-15']) 
print(kakao)
print(kakao['2016-02-19'])

>>
2016-02-19    92600
2016-02-18    92400
2016-02-17    92100
2016-02-16    94300
2016-02-15    92300
dtype: int64
92600
```

```python
for v in kakao.values:
    print(v)

>>
92600
92400
92100
94300
92300
```

```python
mine = Series([10, 20, 30], index = ['lg', 'lotte', 'sk'])
friend = Series([10, 30, 20], index = ['lotte', 'sk', 'lg'])

print(mine)
print(friend)
merge = mine + friend
print(merge)

>>
lg       10
lotte    20
sk       30
dtype: int64
lotte    10
sk       30
lg       20
dtype: int64
lg       30
lotte    30
sk       60
dtype: int64
```
### Pandas DataFrame : 2차원 자료구조

```python
from pandas import DataFrame

raw_data = {'col0': [1, 2, 3, 4], 'col1': [10, 20, 30, 40], 'col2': [100, 200, 300, 400]} 

data = DataFrame(raw_data)
print(data)

>>
   col0  col1  col2
0     1    10   100
1     2    20   200
2     3    30   300
3     4    40   400
```

```python
from pandas import Series, DataFrame 
daeshin = {'open': [11650, 11100, 11200, 11100, 11000], 
               'high': [12100, 11800, 11200, 11100, 11150], 
               'low' : [11600, 11050, 10900, 10950, 10900], 
               'close': [11900, 11600, 11000, 11100, 11050]} 
daeshin_day = DataFrame(daeshin, columns=['open', 'high', 'low', 'close'])
print(daeshin_day)

>>
    open   high    low  close
0  11650  12100  11600  11900
1  11100  11800  11050  11600
2  11200  11200  10900  11000
3  11100  11100  10950  11100
4  11000  11150  10900  11050
```

```python
from pandas import Series, DataFrame 
daeshin = {'open': [11650, 11100, 11200, 11100, 11000], 
               'high': [12100, 11800, 11200, 11100, 11150], 
               'low' : [11600, 11050, 10900, 10950, 10900], 
               'close': [11900, 11600, 11000, 11100, 11050]} 
date = ['16.02.29', '16.02.26', '16.02.25', '16.02.24', '16.02.23']
daeshin_day = DataFrame(daeshin, columns=['open', 'high', 'low', 'close'], index=date)

print(daeshin_day)

>>
           open   high    low  close
16.02.29  11650  12100  11600  11900
16.02.26  11100  11800  11050  11600
16.02.25  11200  11200  10900  11000
16.02.24  11100  11100  10950  11100
16.02.23  11000  11150  10900  11050
```
* open column의 전체 데이터
```python
print(daeshin_day['open'])

>>
16.02.29    11650
16.02.26    11100
16.02.25    11200
16.02.24    11100
16.02.23    11000
Name: open, dtype: int64
```
* 특정 날짜의 전체 데이터
```python
print(daeshin_day.loc['16.02.25'])

>>
open     11200
high     11200
low      10900
close    11000
Name: 16.02.25, dtype: int64
```

* 컬럼과 인덱스 가져오기

```python
print(daeshin_day.columns)
print(daeshin_day.index)

>>
Index(['open', 'high', 'low', 'close'], dtype='object')
Index(['16.02.29', '16.02.26', '16.02.25', '16.02.24', '16.02.23'], dtype='object')
```

## 주식차트 그리기

### pandas-datareader 패키지 설치

* Anaconda prompt에서
* conda install pandas-datareader

### datareader 사용

```python
import pandas as pd 
import pandas_datareader.data as web 
import matplotlib.pyplot as plt 

# Get GS Data from Yahoo 
gs = web.DataReader("078930.KS", "yahoo", "2014-01-01", "2016-03-06") 
new_gs = gs[gs['Volume']!=0]   #거래량 0인것 제외
print(new_gs)
```

```python
new_gs = new_gs.dropna()
```
* csv 파일 만들기
```python
new_gs.to_csv('out.csv', sep = ',')
```

* 이동평균 구하기

```python
ma5 = new_gs['Adj Close'].rolling(window=5).mean() 
ma20 = new_gs['Adj Close'].rolling(window=20).mean() 
ma60 = new_gs['Adj Close'].rolling(window=60).mean() 
ma120 = new_gs['Adj Close'].rolling(window=120).mean() 
```

* Insert Columns

```python
new_gs.insert(len(new_gs.columns), "MA5", ma5) new_gs.insert(len(new_gs.columns), "MA20", ma20) new_gs.insert(len(new_gs.columns), "MA60", ma60) new_gs.insert(len(new_gs.columns), "MA120", ma120)
```

* plot 차트 그리기

```python
import matplotlib.pyplot as plt 

plt.plot(new_gs.index, new_gs['Adj Close'], label='Adj Close')
plt.plot(new_gs.index, new_gs['MA5'], label='MA5') 
plt.plot(new_gs.index, new_gs['MA20'], label='MA20')
plt.plot(new_gs.index, new_gs['MA60'], label='MA60')
plt.plot(new_gs.index, new_gs['MA120'], label='MA120')
plt.legend(loc="best") 
plt.grid() 
plt.show()
```

![](https://github.com/namjunemy/TIL/blob/master/Python/image/plot_chart.PNG?raw=true)

