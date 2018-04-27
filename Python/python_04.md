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

