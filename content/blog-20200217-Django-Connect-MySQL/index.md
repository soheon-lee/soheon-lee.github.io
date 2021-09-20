---
emoji: 👑
title: Django에 MySQL 연결하여 데이터 저장하기
date: '2020-02-17 19:00:00'
author: 이워크
tags: django python 위코드 wecode 장고 MySQL
categories: 개발블로그
---

Django에서는 기본 데이터 베이스로 sqlite3를 제공해준다. setting.py 파일을 보면 sqlite3에 대한 설정을 확인할 수 있다. 아래와 같다.

```python
DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.sqlite3',
    'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
  }
}
```

나는 앞으로 MySQL을 사용할 것이기 때문에, 이 기본 정보를 바꾸고 Django-MySQL 조합을 완성하려 한다.

---

### 우분투 MySQL 설치

```bash
$ sudo apt install -y mysql-server
$ sudo mysql_secure_installation
[NO]
[최고관리자 password]
[Y:익명 사용자 제거]
[N:외부 로그인 허용]
[Y:테스트 데이터베이스 삭제]
[Y:privilege table 다시 로드]
```

### 환경 구성

django가 설치되어 있는 가상환경을 활성화한 뒤, `mysqlclient`를 설치한다. 이 라이브러리를 활용하여 연동할 것이다.

```bash
$ pip install mysqlclient
```

##### 우분투에서 mysqlclient 설치시 오류가 발생한다면,

<p style="background-color:pink; padding:5px">Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-install-zbw18e9_/mysqlclient/  </p>

이런 에러.  
또는,

<p style="background-color:pink; padding:5px">/bin/sh: 1: mysql_config: not found  </p>

이런 에러.  
이때,

```bash
sudo apt-get install libmysqlclient-dev
```

다시

```bash
pip install mysqlclient
```

### MySQL에 데이터베이스 생성

현재 프로젝트에서 이용할 데이터베이스를 MySQL 내부에 만들어주자. starbucks 라는 데이터 베이스를 생성한다.

```sql
mysql> create database starbucks character set utf8mb4 collate utf8mb4_general_ci;
Query OK, 1 row affected (0.01 sec)
```

character set utf8mb4 - 한글 사용 활성화

```sql
mysql> use starbucks
Database changed
mysql> show tables;
Empty set (0.00 sec)
```

지금은 아무 테이블도 없지만, 앞으로 이것 저것 넣으면 된다.

##### &#10068; 데이터베이스 삭제하려면

```
DROP DATABASE <database_name>
```

### Django-MySQL 연동

여태 settings.py에 모든 정보를 몰아서 저장했지만, 지금부터는 my_settings.py라는 새로운 파일에 민감한 정보들을 따로 저장해두고, settings.py에서 import 해서 사용할 것이다.

> <p class="quote" style="max-width: 100%; padding:10px; border-left:solid 3px gray; text-align:left"> my_settings.py 파일은 외부와 공유할 일이 없는 정보이기 때문에 .gitignore이라는 숨김 파일에 이름을 등록하여 github에 업로드 되지 않도록 만들어야 한다.</p>

```bash
vim my_settings.py
```

```python
DATABASES = {
    'default' : {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'starbucks',
        'USER': 'root',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
```

`my_settings.py`파일이 완성되었다. 그럼 진짜 `setting.py`파일에서 import하러 가자!

```bash
vim settings.py
```

```python
import my_settings

DATABASES = my_settings.DATABASES
```

모든 설정을 수정하였다면 데이터베이스에 테이블을 만들어줘야한다. `./manage.py migrate`

```sql
mysql> use starbucks
mysql> show tables;
```

`장고_어쩌구` 세 테이블과, 내가 starbucks_menu.models에 만들어 둔 클래스에 해당하는 표들이 보인다.

---

이제 쓰면 된다.

```toc

```
