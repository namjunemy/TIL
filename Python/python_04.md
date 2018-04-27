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

