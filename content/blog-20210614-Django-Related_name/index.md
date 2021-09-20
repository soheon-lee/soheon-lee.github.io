---
emoji: 👑
title: Django과 Reverse relation과 related_name
date: '2020-06-14 19:00:00'
author: 이워크
tags: django python related_name 위코드 wecode 장고 역참조
categories: 개발블로그
---

# 정참조와 역참조 객체 서로 호출하기

데이터베이스에서 두 테이블이 참조 관계에 있는 경우를 생각해보자. 예를 들어, `User` 테이블과 사용자의 직업인 `Occupation` 테이블이 있다. 두 테이블은 N:1 관계에 있으며, `User` 객체가 `Occupation` 객체를 참조하고 있다. `User` 가 `Occupation` 을 선택하여 입사 원서를 작성한다고 가정해보자.

```python
class User(models.Model):
    name	= models.CharField(max_length = 50)
    job		= models.ForeignKey('Occupation', on_delete = models.CASCADE)
    created_at	= models.DateTimeField(auto_now_add = True)

class Occupation(models.Model):
    name = models.CharField(max_length = 50)
```

`User` 객체는 `Occupation` 객체를 정참조 하고 있으므로, 속성 이름으로 바로 접근 할 수 있다. User1을 선택하여, 그 사람의 job을 찾아보자.

```python
user1 = User.objects.get(id = 1)
user1.job.name
>>> 'Developer'
```

그러나 `Occupation` 객체는 `User` 객체를 역참조 하고 있으므로 바로 접근이 불가능하다. `developer` 이라는 `Occupation`을 가지고 있는 유저를 모두 찾아보자.

```py
job1   = Occupation.objects.get(name = 'developer')
people = job1.user.all() # 이게 될까?
```

❌ 안 됨 ❌

```py
Traceback (most recent call last):
  File "<console>", line 1, in <module>
AttributeError: 'Occupation' object has no attribute 'user'
```

그렇다고 절대로 사용하지 못하는 것은 절대 아니니 걱정하지말자. **역참조 관계에 있을 때는 `[classname]_set` 이라는 속성을 사용하여 접근**해야한다.

```py
job1   = Occupation.objects.get(id = 1)
people = job1.user_set.all()
>>> <QuerySet[<Object User Object(1)>, <Object User Object(2)>]>
```

이 때, `user_set` 대신 사용할 수 있는 것이 `related_name`이다. 역참조 대상인 `user` 객체를 부를 이름.

> 즉, `User` 클래스를 정의할 때, 정참조 하고 있는 `Occupation` 클래스의 인스턴스에서 어떤 명칭으로 거꾸로 호출당할 지 정해주는 이름인 것이다.

# What related names do

앞의 예시를 다시 보자. 아무래도 `Occupation`의 입장에서는 입사 지원자들을 `appliers`라고 부르는 것이 더 직관적이고 편할 것 같다.

```python
class User(models.Model):
    name = models.CharField(max_length = 50)
    job	 = models.ForeignKey(
            'Occupation',
            on_delete    = models.CASCADE,
            related_name = 'appliers' ------------------------- [Key Point !]
        )
    created_at	= models.DateTimeField(auto_now_add = True)

class Occupation(models.Model):
    name = models.CharField(max_length = 50)
```

**[Key Point]** 를 눈여겨 보자.
`User`객체를 정의할 때, `job`이라는 속성에 `Occupation`객체가 연결되어 정참조하고 있다. `Occupation`객체의 인스턴스와 연결되어 있는 `User` 객체를 거꾸로 불러올 때, `appliers` 라는 이름으로 부르기 위해 `job` 속성에 `related_names = 'appliers'`를 함께 지정해주었다.

```py
job1   = Occupation.objects.get(id = 1)
people = job1.appliers.all()
>>> <QuerySet[<Object User Object(1)>, <Object User Object(2)>]>
```

잘 동작하는 것을 알 수 있다. 모든 Foreign Key에 related_name을 붙여줄 필요는 없다. 때에 따라, 참조하고 있는 객체 이름에 `_set`을 붙이는 것이 더 직관적인 경우가 굉장히 많기 때문이다.

# Related name이 필수인 경우가 있다.

바로 한 클래스에서 서로 다른 두 컬럼(속성)이 같은 테이블(클래스)를 참조하는 경우이다.
앞서 설명한 상황에서, 지원자가 필수로 신청한 `occupation`외에, 2지망인 `occupation`도 받는다고 가정해보자.

```python
class User(models.Model):
    name       = models.CharField(max_length = 50)
    job	       = models.ForeignKey('Occupation', on_delete = models.CASCADE)
[*] choice_2nd = models.ForeignKey('Occupation', on_delete = models.CASCADE, null = True)
    created_at = models.DateTimeField(auto_now_add = True)

class Occupation(models.Model):
    name = models.CharField(max_length = 50)
```

> - 참고로 위와 같은 선언은 애초에 마이그레이션이 되지 않는다. related_name 지정하라는 문구만 뜸.

`User`객체에서 `Occupation`객체를 정참조 하는 속성이 두 개이다. 다시 말해 `developer`이라는 `Occupations`객체의 인스턴스를 1지망으로 선택한 지원자와 2지망으로 선택한 지원자가 따로 구별되어있다는 뜻이 된다. 아래 두 인스턴스를 보자.

```py
user1 = User.objects.create(name = 'Nick', job_id = 1) #developer
user2 = User.objects.create(name = 'Sue', job_id = 2, choice_2nd_id = 1)
```

`user1`은 1지망은 `job`으로 `id`가 `1`번인 `developer`이다.
`user2`의 1지망은 `2`번 `job`이고, **2지망**이 `developer`이다.

### 이 때 related_name이 없다면?

```
job1 = Occupation.objects.get(id = 1)
job1.user_set.all()
```

의 결과가 생성될 수 있을까?
❌ 안 됨 ❌

> `Occupation`객체를 정참조 하고 있는 컬럼이 `job`과`choice_2nd`두 개이므로, 그저 user_set이라는 속성만으로는 자신을 바라보고 있는 두 User 객체 가운데 어떤 속성에 접근해야할 지 알 수가 없기 때문이다. 즉, `developer`을 1지망으로 고른 사람들의 목록(`Nick`)을 가져와야할 지, 2지망으로 고른 사람들의 목록(`Sue`)을 가져와야할 지 알 수가 없기 때문이다.

### 바로 이럴 때 related_name이 필수인 것이다.

```python
class User(models.Model):
    name = models.CharField(max_length = 50)
    job	 = models.ForeignKey(
            'Occupation',
            on_delete    = models.CASCADE,
            related_name = 'appliers' ------------------------- [Key Point !]
        )
    choice_2nd  = models.ForeignKey(
            'Occupation',
            on_delete    = models.CASCADE,
            null         = True
            related_name = 'second_appliers'
        )
    created_at	= models.DateTimeField(auto_now_add = True)
```

**이제는 `developer`을 1지망으로 지원한 Nick과 2지망으로 지원한 Sue를 구분하여 호출할 수 있다.**

```py
job1 = Occupation.objects.get(id = 1)

job1.appliers.all()
>>> <QuerySet[<Object User Object(1)>]> # ---> Nick

job1.second_appliers.all()
>>> <QuerySet[<Object User Object(2)>]> # ---> Sue
```

> #### 예시에서는 Foreign Key만을 다루었는데, ManyToMany 관계에 있을 때에도 related_name은 같은 원리로 동작한다. 헷갈리지 않게 주의하자.

# 마치며

웹의 구조나 서비스가 복잡해질 수록, 클래스 사이의 참조가 많아진다. 일대다는 물론이고, 다대다 관계도 계속 늘어난다. 그럴 때일 수록 related name이 중요하다고 생각한다. 어떻게든 migration만 되면 되지. 라는 생각으로 related_name을 마음대로 설정하다 보면, 나중엔 변수 이름을 아무렇게나 정했을 때만큼이나 의미를 알수 없는 코드를 양산하게 되기 때문이다.

오늘도 많이 배웠다.

```toc

```
