---
emoji: 👑
title: Django로 query string 받기
date: '2020-02-29 19:00:00'
author: 이워크
tags: django python 위코드 wecode 장고 querystring
categories: 개발블로그
---

# 쿼리스트링

프론트엔드에서 요청을 받을 때, 우리 백엔드는 end point 주소를 받는다. 엔드포인트를 보고, '아, 이런 정보를 얻고싶구나' 하고 알 수 있다. 하지만 거의 같은 자료인데 조금씩 다른 자료에게 모두 각자 다른 url을 부여할 필요는 없다.  
쿼리스트링을 사용하여 url의 정보를 parameter로 받으면 같은 url, 같은 view 만으로도 입력한 parameter의 정보에 따라 <span style="background-color:beige; padding:2px">**필요한 자료만 제공**</span>하면 되는 것이다.

## 어떻게 생겼나?

유투브 채널에 접속해보았다. BTS의 뮤직비디오다.  
https://www.youtube.com/watch?v=mPVDGOVjRQ0  
www.youtube.com/watch라는 url 뒤에 **?v=mPVDGOVjRQ0** 라는 부분이 보인다.  
이렇게 물음표 뒷부분을 Query String 이라고 부른다. **`v`**라는 변수에 **`mPVDGOVjRQ0`** 라는 값을 담는 것이다.

## 사용하기

설명 그대로 물음표 뒤에 변수와 값을 담는다. 여러개를 담고 싶을 때는 **`&`**를 사용한다.  
**`?name=sophia&age=28&gender=F`**  
view에서 불러올 때는  
**`my_name = request.GET.get('name')`**  
**`my_age = request.GET.get('age')`**  
예시로 다시 보자.

#### 예시 1. 카테고리에 따른 제품 목록 나타내기

베스킨 라빈스 웹 페이지에는 '아이스크림', '아이스크림 케이크', '음료', '커피', '디저트' 라는 카테고리에 따라 제품을 분류한다. 각 제품을 표출하는 view를 일일이 만들 필요는 없다. type을 지정해주고, 해당하는 것만 불러오면 되니까.

**참고로 변수 뒤에 값의 자료형은 string이든 integer든 상관없다.**

그럼 <span style="color:orange; background-color:beige">views.py</span>를 작성해보자

- query string을 문자열로 받을 때

```
baskinrobbins/product/menu?type=아이스크림
baskinrobbins/product/menu?type=아이스크림케이크
baskinrobbins/product/menu?type=음료
baskinrobbins/product/menu?type=커피
baskinrobbins/product/menu?type=디저트
```

```python
class MenuView(View):
      def get(self, request):
        menu_name = request.GET.get('type', None)
        products  = Product.objects.filter(menu = menu_name).values()

        return JsonResponse({'products':list(products)}, status = 200)
```

코드에서 menu_name 선언할 때 `None`을 지정해주는 이유는, 값이 들어오지 않았을 때를 대비한 것이다.

<span style="color:orange; background-color:beige">urls.py</span>도 한 줄 이면 된다.

```python
urlpatterns = [
    path('/menu', MenuView.as_view()),
]
```

#### 예시 2. 도시와 지역구 선택했을 때 매장 목록 나타내기

- query string을 숫자로 받을 때

```
baskinrobbins/store?city=1&district=20
baskinrobbins/store?city=2&district=44
baskinrobbins/store?city=3&district=60
```

```python
class StoreSearchView(View)
      def get(self, request):
        city       = request.GET.get('city', None)
        district  = request.GET.get('district', None)
        stores    = Store.objects.filter(city_id='city', district_id='district').values()

        return JsonResponse({'stores':list(stores)}, status = 200)
```

# 만약 안쓴다면...?

진짜 일일이

```python
class IcecreamView(View):
  def get(self, request):
    products = Product.objects.filter(menu = '아이스크림').values()
    return JsonResopnse({'products':list(products)}, status = 200)

class IcecreamCakeView(View):
  def get(self, request):
    products = Product.objects.filter(menu = '아이스크림케이크').values()
    return JsonResopnse({'products':list(products)}, status = 200)

class CoffeeView(View):
  def get(self, request):
    products = Product.objects.filter(menu = '커피').values()
    return JsonResopnse({'products':list(products)}, status = 200)

...
```

url도 줄줄이

```python
urlpatterns = [
  path('/menu/icecream', IcecreamView.as_view()),
  path('/menu/icecream_cake', IcecreamCakeView.as_view()),
  path('/menu/coffee', CoffeeView.as_view()),
  ...
]
```

어우 못해못해.  
 query string 짱짱맨 !

---

```toc

```
