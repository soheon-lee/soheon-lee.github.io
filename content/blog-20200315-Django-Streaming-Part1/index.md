---
emoji: 👑
title: Django로 스트리밍 하기 part1 - binary streaming
date: '2020-03-15 19:00:00'
author: 이소헌
tags: python 위코드 wecode Django streaming
categories: 개발블로그
---

# 사전 연습 - text streaming

음악 스트리밍 웹인 TIDAL을 클론하기 위해서는 음악 스트리밍 기능을 구현하는 것이 가장 중요하다. 음악 스트리밍을 위해서 사전 연습으로 우선 텍스트 스트리밍을 먼저 해보려고 한다.

과정은 아래와 같다.

1. 파일 이름을 path parameter로 받기
2. 해당하는 파일을 불러내기
3. 그 파일 속에 문구를 streaming으로 보내기

## 파일 준비

내가 준비한 파일은 text_sample이라는 단순 텍스트 파일이다. 파일 내부에는 볼빨간 사춘기 - 별 보러 갈래 의 가사가 담겨있다.

<details markdown="1">
  <summary> text_sample </summary>
  Maybe it's like a dream  
  I see the stars over me  
  Maybe it's like a magic  
  I know you, you, you're my star  
  Saturday night 재미없는 얘기  
  No beer, no cheers 우리 둘만 여기  
  재미없는 사람끼리 눈이 맞았나 봐  
  You've heard of my songs  
  어떤 별을 가장 좋아하냐며  
  미소를 띠고 내게 말해  
  별 보러 갈래?  
  Listen to our favorite songs  
  좋아하는 노랠 듣고  
  웃고 떠들다 보면  
  We drive away  
  어느새 멋진 바다  
  위로 별들이 쏟아져내려  
  Can't you see the stars?
  They called it milky way 쏟아져 머리 위로
  넌 나를 업고 모래사장을 뛰어다녀, yeah
  그중 가장 예쁜 저 별을 찾아서
  밤이 새도록 뛰어다니고
  Stars are over me
  Maybe I know the name
  I see the stars over me
  Maybe you got a planet
  I know you, you got my star
  Everyday night 매일 같은 얘기
  No feeling, no chilling 우리 둘만 여기
  재미없는 사람끼리 눈이 맞았나 봐
  You've heard of my songs
  어떤 별을 가장 좋아하냐며
  미소를 띠고 내게 말해
  별 보러 갈래?
  Listen to our favorite songs
  좋아하는 노랠 듣고
  웃고 떠들다 보면
  We drive away
  어느새 멋진 바다
  위로 별들이 쏟아져내려
  Can't you see the stars?
  모래 위에 누워서
  저 별을 다 세보다가
  아름다웠던 우리 이 순간을
  저 별에 담아서
  We fell in love
  See the star
  Be your star, yeah, yeah
  They called it milky way 쏟아져 머리 위로
  넌 나를 업고 모래사장을 뛰어다녀, yeah
  그중 가장 예쁜 저 별을 찾아서
  밤이 새도록 뛰어다니고
  Stars are over me
</details>

## views.py 내부 로직

```python
[1] from django.http  import StreamingHttpResponse

    class StreamTextView(View):
      def get(self, request):
        try:
          in_file = request.GET.get('file_name', None)
[2] ---   stream    = self.iteration(in_file)
[3] ---   response  = StreamingHttpResponse(stream, status = 200, content_type = 'text/event-stream')
          response['Cache-Control'] = 'no-cache'

          return response

        except Exception as identifier:
          print(identifier)

[4]   def iteration(self, in_file):
        with open(in_file, 'rb+') as f:
          while True:
            c = f.read()
            if c:
[5] ---       yield c
            else:
              break
```

- [1] HttpResponse, JsonResponse 외에도 **StreamingHttpResponse** 를 import 한다.
- [2] query string 으로 받은 in_file에서 내부 내용을 가져오는 과정을 처리한다.

- [3] 처리해서 받은 내부 내용을 **response**에 담아 return

**[2]의 과정을 좀 더 자세히 보자.**

- iteration이라는 함수는 [4]에 있는데, in_file이라는 파일을 f 라는 이름으로 연다.
- c = f.read()로 읽은 문장을 할당한다.
- 문장 c 가 있으면 `yield c`
- yield 한 문장이 [2] stream 변수에, 분할되어 (스트리밍되어) 할당되는 것이다.

## yield?

우선 `return`과 마찬가지로 함수의 결과를 반환하는 역할을 한다. 그렇다면 차이점은 무엇일까?

> **<span style="color:darkblue; background-color:beige; padding:2px;">Return</span>** sends a specified value back to its caller whereas **<span style="color:green; background-color:beige; padding:2px;">Yield</span>** can produce a sequence of values. We should use yield when we want to iterate over a sequence, but don’t want to store the entire sequence in memory. <a href="https://www.geeksforgeeks.org/use-yield-keyword-instead-return-keyword-python/">reference</a>

즉, return은 결과를 한 번에 반환하고, yield는 결과의 시퀀스를 반환하여, 결과를 순회하여 응용할 때 사용하면 된다. 따라서, yield를 사용하면 파이썬 함수를 제너레이터로 사용할 수 있게 된다.

스트리밍은, **결과를 연쇄적으로 반환하는 과정**이기 때문에 yield를 쓰는 것이다.

## 결과

실행시켜보자

```bash
http -v 10.58.2.53:8001/music/text?file_name=text_sample
```

```bash
Maybe it's like a dream
I see the stars over me
Maybe it's like a magic
I know you, you, you're my star
Saturday night ì¬ë¯¸ìë ì
                           ë¡ ë³
                                 ë¤ì´ ìì
                                            ì ¸ë´ë ¤
Can't you see the stars?
They called it milky way ìì
                             ì ¸ ë¨¸ë¦¬ ì
                                          ë¡
ë
  ë ì  ë³
            ì
               ì°¾ì
                    ì
                     
ë°¤ì´ ìë
          ë¡ ë°ì´ë¤ëê³ 
Stars are over me
Maybe I know the name
I see the stars over me
Maybe you got a planet
I know you, you got my star
Everyday night ë§¤ì¼ ê°ì ì
                              ë¡ ë³
                                    ë¤ì´ ìì
                                               ì ¸ë´ë ¤

```

한글이 왜 이렇게까지 깨지는 지는 참 의문이다 ;;

# 본편 - 음악 파일

다행히, text file을 음악 파일로 바꿔주기만 하면 된다.  
대신 쿼리 스트링으로 음악 파일이 아닌 음악의 track_id를 받아와서, 그 track의 url로 연결지어 줄 것이다.

## views.py

```python
class StreamTextView(View):
    def get(self, request):
        try:
            track_id    = request.GET.get('track_id', None)
            track       = Track.objects.get(id = track_id)
            music_url   = track.music_url
            stream      = self.iteration(music_url)
            response    = StreamingHttpResponse(stream, status = 200, content_type = 'text/event-stream')
            response['Cache-Control'] = 'no-cache'

            return response
        except Exception as identifier:
            print(indentifier)

    def iteration(self, music_url):
        with open(music_url, 'rb+') as f:
            while True:
                c = f.read()
                if c:
                    yield c
                else:
                    break
```

## 결과

실행시켜 보자

```bash
http -v 10.58.2.53:8001/music/text?track_id=1
```

```bash
GET /music/text?track_id=1 HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: 10.58.2.53:8001
User-Agent: HTTPie/2.1.0-dev



HTTP/1.1 200 OK
Cache-Control: no-cache
Connection: close
Content-Type: text/event-stream
Date: Sun, 15 Mar 2020 13:37:17 GMT
Server: WSGIServer/0.2 CPython/3.7.4
Vary: Origin
X-Content-Type-Options: nosniff
X-Frame-Options: DENY



+-----------------------------------------+
| NOTE: binary data not shown in terminal |
+-----------------------------------------+

```

## ????

바이너리 파일을 터미널에서 볼 수 없다고 ??

## 해결

```bash
http -v 10.58.2.53:8001/music/text?track_id=1 > a

xxd a
xxd -b a
```

이러면 된다 ~

```bash
00113ff0: 2240 33c5 07b6 18e4 be3c ce1f 82f5 b8ee  .@3......<......
00114000: 33dc dda5 d286 eec1 da0b b747 7afb 6792  3..........Gz.g.
00114010: 3207 51df 8021 d97e e451 66e7 1383 db78  2.Q..!.~.Qf....x
00114020: 02cc 6a0e 8252 b149 ca1a e261 c3cc b3b2  ..j..R.I...a....
00114030: f9eb 52d8 f41d 3300 f6f7 cb7f e8a2 561c  ..R...3.......V.
00114040: 7bfc 4874 5375 dafb 0347 f975 1d34 bf17  {.HtSu...G.u.4..
00114050: deac d4b6 3f94 426a 397f b2aa b24b 13dd  ....?.Bj9....K..
00114060: 0843 a9c5 d8cb 2725 f108 729a 3135 3365  .C....w%..r.153e
00114070: f6a7 a94d 7e53 4b86 78fe afd0 d9b9 6a8a  ...M~SK.x.....j.
00114080: 1f95 ddaf da96 2c75 f88d 2e46 0916 600a  ......,u...F..b.
00114090: e21e 6190 23f2 b0ec 9e05 676e ffff ffe0  ..a.#.....gn....
001140a0: 40c4 bbd6 0000 6e09 8044 0a60 46e3 4514  @.....n..D..F.E.
001140b0: 8931 2921 a89b 82da d05a ec0d 9032 f03c  .1)!.....Z...2.<
001140c0: 483a 0116 1734 2c12 0c23 9960 0a9a a051  H:...4,..#.`...Q
001140d0: 0018 3c01 0003 8680 9181 8587 ed9c 04a8  ..<.............
001140e0: 90b8 808a 7397 bc15 1ab8 5410 7cd3 0444  ....s.....T.|..D
001140f0: be31 9553 30e4 4687 5e69 8984 80d7 31d8  .1.S0.F.^i....1.
00114100: 8042 0597 a243 1b7f 648c 8d60 9219 a734  .B...C..d......4
00114110: 976e 1c7d e34b b16a 060b 6772 a809 f3ca  .n.}.K.j..gr....
00114120: 8a18 6dda b3b7 2ad4 adef 985c 8f24 3701  ..m...*....\.$7.
00114130: 5d9d 87e3 709a 96ea c5e5 1f04 6715 5d0e  ]...p.......g.].
00114140: 7431 1853 7913 b96e 4953 1d53 ea17 279f  t1.Sy..nIS.S....
00114150: a3a2 a09d 92c3 f632 75e6 3d73 aa79 e6de  .......2u.=s.y..
00114160: 65af c429 e293 ecbe 3120 dbf9 25ff fbe0  e..)....1 ..%...
```

**10.58.2.53:8001/music/text?track_id=1**  
이걸 chrome 주소창에 입력하면, 크롬 브라우저가 binary를 읽어서 파일을 재생시켜줌 !

---

```toc

```
