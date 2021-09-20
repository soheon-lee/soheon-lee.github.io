---
emoji: 👑
title: With as로 열 수 있는 파이썬 객체 생성하기
date: '2020-04-05 19:00:00'
author: 이소헌
tags: python 위코드 wecode withAs
categories: 개발블로그
---

파이썬으로 다양한 객체들을 다루다보면 종종 `with open`을 사용한다.  
특히 파일을 열 때 유용하게 사용하는데, 텍스트(.txt) 파일을 활용하여 작업할 때는 물론이며 음악 파일을 다룰 때에도 어김없이 등장한다.

- <a style="color:orange; text-decoration:none; font-weight:800" href='https://soheon-lee.github.io/blog-20200315-Django-Streaming-Part1/'> Django로 음악 스트리밍하기 PART1</a>
- <a style="color:orange; text-decoration:none; font-weight:800" href='https://soheon-lee.github.io/blog-20200315-Django-Streaming-Part2/'> Django로 음악 스트리밍하기 PART2</a>

현재 진행중인 프로젝트에서는 database 로 연결되는 connection을 열고 닫을 때도 사용하고 있다.  
그렇다면 이 기능의 정체는 무엇일까? 나만의 데이터도 with open으로 열 수 있을까?

# Python 'with' ?

파이썬을 사용하는 사람이라면, 아래 구문이 꽤나 익숙할 것이다. with은 쉽게 말해 아래 구문을 **한 단어로 합쳐놓은 것**이다.

```python
set things up
try:
  do something
finally:
  tear things down
```

여기서 `set things up` 구문은 파일을 여는 등의 명령인데, 열고 나서 `something`을 하고, 그 명령이 끝나고 나면, 결과 여부에 상관없이 (suceeded or not) `tear things down` 하라는 구문이다.  
예를 들어, python에서 mysql.connector 라는 커넥터를 이용하여 데이터베이스와의 연결을 연다고 가정해보자.  
(mysql에 관한 아래 설정은 무시해도 좋다.)

```python
db_connection = mysql.connector.connect
  (
    'database'=DATABASE['database'],
    'user'=DATABASE['root']
  ) -------------------------------------------------- [1]
try:
  db_cursor = db_connection.cursor() ----------------- [2]
finally:
  db_cursor.close() ---------------------------------- [3]

```

1. 데이터베이스 연결을 만든다 (연다).
2. 커서를 연결한다.
3. 커서를 닫는다.

위의 코드는 파일을 열고 어떤 명령을 내리고 저걸 다시 닫는 단순 과정이다. 딱!! 함수화하기에 적당한 과정이 아닌가?

### 이 모든 과정을 한 번에 처리하기 위해 존재하는 것이 'with'이다.

# 객체 생성

파이썬의 거의 모든 자료형은 **객체**다. 어떤 객체를 `with`으로 열고 싶다면, 그 객체의 기본 메소드로 `__enter__`과 `__exit__`이 필요하다.

### \_\_enter\_\_ : 객체가 열리자마자 실행

### \_\_exit\_\_: 객체가 닫힐때 실행

```python
class contolled_execution:
  def __enter__(self):
    set things up
    return thing

  def __exit__(self, type, value, traceback):
    tear things down

with controlled_execution() as thing:
  some code using thing
```

`with`구문이 실행되면, **context manager**이 자동으로 `__enter__` 메소드로 정의된 동작을 수행하고, 이 메소드의 결과로 반환하는 `return`값을 `as`를 사용해 things에 저장한다. Python이 코드의 다른 body 부분을 수행한 다음에는 그 결과가 어떠하든 (finally) `__exit__` 메소드를 실행하게 된다.  
앞서 예시로 든 mysql database connection 에 이 방법을 적용해보자.

```python
class DatabaseConnection():
    def __enter__(self):
      self.cursor = self.db_connection.cursor(buffered=True, dictionary=True)
      return self.cursor

    def __exit__(self, exc_type, exc_value, exc_trance):
    self.cursor.close()

database_connection_cursor = DatabaseConnection()
with database_connection_cursor as db_cursor:
    db_cursor.do_something
```

# file 객체는 어떤 모양인가?

우리가 자주 with open 구문으로 사용하는 file 객체도 사실 뜯어보면 `__enter__`과 `__exit__`이 있다.

```python
>>> f = open("x.txt")
>>> f
<open file 'x.txt', mode 'r' at %%%%>
>>> f.__enter__()
<open file 'x.txt', mode 'r' at %%%%>
>>> f.__exit__(None, None, None)
>>>
```

그래서 우리가 with file open을 사용할 수 있었다.

---

이 방법을 사용하면 다양한 객체를 응용해서 사용할 수 있다. 하지만 이 방법... pymysql 모듈 쓰면 다 알아서 해준다... 있는 자료 잘 활용하는 법도 터득해보자 ..ㅎ...

```toc

```
