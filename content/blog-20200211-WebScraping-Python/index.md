---
emoji: 👑
title: 파이썬으로 웹 스크랩핑 쉽게 하기
date: '2020-02-11 19:00:00'
author: 이워크
tags: django python 위코드 wecode 장고 MySQL beautifulsoup scraping
categories: 개발블로그
---

파이썬 라이브러리를 활용해서 간단한 웹 페이지 스크래핑을 해보자.

## 0. 웹 스크래핑

먼저, 웹 스크래핑이 무엇인지 간단히 짚고 넘어가자. 크롤링과는 어떻게 다른가?
웹 스크래핑은 신문이나 잡지 스크랩처럼 웹 사이트에서 원하는 정보를 따로 추출하여 저장해두는 것을 말한다.  
엄청난 분량의 웹 문서를 사람이 일일이 모으는 일이 거의 불가능하기에 이를 자동으로 해주는 작업이다.
모든 순간 모든 자료를 모두 업데이트하지는 않고, 그 순간의 데이터를 저장할 수 있다.

이 외에도 데이터가 업데이트될 때마다 혹은 주기적으로 특정 시간마다 데이터를 수집하는 과정을 반복하여 업데이트 된 정보를 추출해올 수도 있다. 이런 프로그램</span>을 **웹 크롤러** 라고 한다.

- Billboard chart 200
- Netflix FAQ list

두 페이지에서 웹 스크래핑을 실습해보려한다. 다시 말해, 두 페이지에서 원하는 정보를 가져오려고 한다.

## 1. 라이브러리 설치 & 간단 소개

**`bs4`**와 **`sqlalchemy`**, **`sqlalchemy`** 세 라이브러리가 필요하다. 혹시 모르니 가상환경 아래에서 설치해주었다.

```bash
conda creat -n "scrap01"  # 가상환경 생성
conda activate scrap01    # 가상환경 활성화

pip install bs4           # bs4 설치
pip install requests      # requests 설치
pip install sqlalchemy    # sqlalchemy 설치
```

각 모듈의 역할을 아주 간단히 기술해보자.

| name                                                | function                                                                                                 |
| :-------------------------------------------------- | :------------------------------------------------------------------------------------------------------- |
| **<span style="color:orange">requests</span>**      | Python에서 HTTP 요청을 보내는 모듈. 웹에서 정보를 가져오려면, 웹에 요청을 보내고 그 응답을 받아야하니까. |
| **<span style="color:orange">BeautifulSoup</span>** | html에서 데이터를 추출할 때 사용할 모듈.                                                                 |
| **<span style="color:orange">sqlalchemy</span>**    | 데이터베이스를 만들어주는 역할.                                                                          |

두루뭉실해도 이정도로 이해하고 과정을 진행해가며 조금씩 이해 범위를 넓혀보자.

## 2. 웹 기본 정보 가져오기

먼저 파이썬 파일을 열어서 설치한 라이브러리와 모듈을 불러온다.

```python
# filename : billboard-scraping.py
import sys
import requests

from bs4            import BeautifulSoup
from sqlalchemy     import *
from sqlalchemy.sql import *
from sqlalchemy.orm import relationship, sessionmaker
from sqlalchemy.ext.declarative import declarative_base
```

큰 어려움 없이 import import import !  
가장 기본적인 정보인 홈페이지 url을 가져오고, 페이지를 구성하는 html을 뽑아내보자.  
빌보드 웹페이지에 접속하여 'BILLBOARD 200' 리스트가 있는 페이지의 url을 알아낸다.

> 이것이다.  
> **https://www.billboard.com/charts/billboard-200**

```python
req = requests.get('https://www.billboard.com/charts/billboard-200')
# get 해오기
html = req.text
# get 해온 내용의 'text'를 가져온다
soup = BeautifulSoup(html, 'html.parser')
# html 소스를 파이썬으로 불러옴 (parse)
```

이렇게, 기초 작업이 되었다. **<span style="color:green">soup</span>**이라는 변수에 빌보드차트 200 홈페이지를 구성하는 모든 html 구조가 할당되어있다. 이 가운데 우리가 원하는 요소를 콕 집어서 가져올 것이다.

## 3. 원하는 요소(정보) 가져오기

빌보드 차트에서 내가 원하는 자료는 아래 네가지다.

1. 순위
2. 곡 이름
3. 가수 이름

먼저 빌보드 웹페이지 개발자도구에서, 내가 원하는 자료의 요소(element)를 택한다.  
**개발자 도구 > 요소 선택 > 태그 우클릭 > copy > copy selector**

![개발자도구-요소선택](./image/IMG_0490.jpg)

복사한 정보는 아래.
`#charts > div > div.chart-list.container > ol > li:nth-child(1) > button > span.chart-element__information > span.chart-element__information__song.text--truncate.color--primary`  
이렇게나 길다니  
이 정보를 아래 코드 안에 argument로 넘겨줄 것이다.

```python
rank            = soup.select('') # 순위
song            = soup.select('') # 곡 이름
singer          = soup.select('') # 가수 이름
```

```python
song = soup.select('#charts > div > div.chart-list.container > ol > li:nth-child(1) > button > span.chart-element__information > span.chart-element__information__song.text--truncate.color--primary')
```

확인할 겸 프린트를 해본다 (`print(song)`). 결과는 아래.

```python
[<span class="chart-element__information__song text--truncate color--primary">Please Excuse Me For Being Antisocial</span>]
```

우선 리스트가 도출되었다는 것을 알 수 있다. 당황스러운 결과가 표시되긴 하지만, 노래 제목인 **Please Excuse Me For Being Antisocial**이 보이니 반갑다.  
당황스러운 결과에서 우리가 원하는 'text'만 가져와야한다. song이 리스트인 것을 다시 한 번 인식하자.

```python
song[0].text
```

결과는 ?  
<span style="background-color:pink">Please Excuse Me For Being Antisocial </span>  
Oh Yeah &#128571; 노래 이름이 별로 마음에 들진 않지만 어쨌든 1위곡이다. 그런데 우리가 필요한 것은 1위곡 뿐만 아니라, 차트 200에 올라있는 모든 곡의 이름이다. 여기서 우리가 복사해 온 셀렉터의 이름을 다시 한 번 보자.  
<span style="background-color:lightgray">\#charts > div > div.chart-list.container > ol > </span><span style="background-color:orange">li:nth-child(1)</span><span style="background-color:lightgray"> > button > span.chart-element\_\_information > span.chart-element\_\_information\_\_song.text--truncate.color--primary'</span>

의심스러운 친구를 발견한다. <span style="background-color:orange">li:nth-child(1)</span>로 리스트 하나만 선택한 것이다. 리스트 내부에 모든 요소를 선택하려면<span style="background-color:orange"> li </span>로 바꾸면 된다.  
rank와 singer도 같은 방법으로 가져온다.

<details markdown="1">
<summary>&#127774; 지금 까지 코드</summary>

```python
  import sys
  import requests

  from bs4            import BeautifulSoup
  from sqlalchemy     import *
  from sqlalchemy.sql import *
  from sqlalchemy.orm import relationship, sessionmaker
  from sqlalchemy.ext.declarative import declarative_base

  req = requests.get('https://www.billboard.com/charts/billboard-200')
  html = req.text
  soup = BeautifulSoup(html, 'html.parser')

  rank    = soup.select('#charts > div > div.chart-list.container > ol > li > button > span.chart-element__rank.flex--column.flex--xy-center.flex--no-shrink > span.chart-element__rank__number')
  song    = soup.select('#charts > div > div.chart-list.container > ol > li > button > span.chart-element__information > span.chart-element__information__song.text--truncate.color--primary')
  singer  = soup.select('#charts > div > div.chart-list.container > ol > li > button > span.chart-element__information > span.chart-element__information__artist.text--truncate.color--secondary')
```

</details>

## 4. 저장할 데이터 베이스 살펴보기

html 코드에서 자료를 콕 집어 가져왔다면, 그 자료를 따로 저장할 데이터베이스가 필요하다. 보통, 자료를 가져오기 전에 먼저 모델(표)을 만들어둔다. 해야할 일을 아래와 같다.

1. 저장할 데이터베이스와 내부 표(모델) 만들기
2. 뽑아온 자료 표에 저장하기 (rank song singer)

#### 4-1. 데이터베이스 만들기

```python
engine = create_engine('sqlite:///billboard_chart_200.db')
Base   = declarative_base()

class Music(Base):
  __tablename__ = 'musics'
  id = Column(Integer, primary_key=True)
  rank = Column(String(50))
  song = Column(String(50))
  singer = Column(String(50))

Music.__table__.create(bind=engine, checkfirst=True)
Session = sessionmaker(bind=engine)
session = Session()
```

실행하면 디렉토리 안에 billboard_chart_200.db 라는 데이터베이스가 만들어진다. 거기에 저장할 Music이라는 테이블을 생성하였다. 클래스 형태로 만들고, 그 내부 속성으로 우리가 넣고싶은 정보의 이름을 넣는다. 이 이름이 컬럼의 이름이 될 것이다.  
아래 세 줄은 우선은 그냥 따라했음.

#### 4-2. 저장하기

```python
music_chart = []

for item in zip(rank, song, singer):
  music_chart.append(
  {
    'rank'  : item[0].text,
    'song'  : item[1].text,
    'singer': item[2].text,
  })

for element in music_chart:
  result = Music(
    rank = element['rank'],
    song = element['song'],
    singer = element['singer'],
  )
  session.add(result)
  session.commit()

request = session.query(Music).all()

for row in request:
  print(row.rank,'|', row.song,'|' ,row.singer)
```

끝! 디렉토리를 보면

> billboard-scraping.py  
> billboard_chart_200.db

두 파일이 포함되어 있고, billboard_chart_200.db에는 내 musics 표가 저장되있다. 확인하는 방법은 아래.

```bash
sqlite3 billboard_chart_200.db
```

```mysql
>>> .tables
>>> select * from musics;
```

---

### Netflix FAQ 스크래핑하기

<details markdown="1">
  <summary>&#127774; Netflix FAQ scraping code</summary>

```python
import requests
import sys
from bs4 import BeautifulSoup

from sqlalchemy import *
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship, sessionmaker
from sqlalchemy.sql import *
e = sys.exit

engine = create_engine('sqlite:///netflix.db')
Base = declarative_base()

class FAQ(Base):
    __tablename__ = 'FAQs'
    id = Column(Integer, primary_key=True)
    question = Column(String(50))

FAQ.__table__.create(bind=engine, checkfirst=True)
Session = sessionmaker(bind=engine)
session = Session()

req = requests.get(
    'https://www.netflix.com/kr/'
)
html = req.text
soup = BeautifulSoup(html, 'html.parser')

faq = soup.select(
    '#faq > div.our-story-card-text > ul > li > button'
)

FAQ_list = []

for item in faq:
    FAQ_list.append(
    {
        'question' : item.text
    })

for element in FAQ_list:
    print('question : ',element['question'])
    result = FAQ(
        question = element['question'],
    )
    session.add(result)
    session.commit()

request = session.query(FAQ).all()
for row in request:
    print(row.name,'|', row.song,'|' ,row.singer)
```

</details>

<details markdown="1">
  <summary>&#127772; Netflix FAQ 저장 결과</summary>

```bash
sqlite3 netflix.db
.tables
select * from FAQs;
```

![netflix_db_img](./image/netflix_faq.png)

</details>

---

현재 우리가 beautifulsoup으로 가져올 수 있는 데이터는 html에 있는 요소에 접근해오기 때문에,
React로 만들어져 있는 등 javascript 렌더링 결과가 표출되는 웹페이지에서는 지금 상태로는 데이터를 추출해올 수 없다.
**SELENIUM** 과 같은 도구로 사용자의 동적 반응을 추가해준 뒤 추출해오는 방법을 추천한다.

```toc

```
