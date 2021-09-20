---
emoji: 👑
title: Django로 스트리밍 하기 part2 - 중간 재생과 Unit test
date: '2020-03-15 21:00:00'
author: 이워크
tags: python 위코드 wecode Django Streaming
categories: 개발블로그
---

# 중간부터 스트리밍

이전에 작성했던 스트리밍 코드는 아래와 같다.

### views.py

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

프론트에 fetch url 을 주고, 그 URL이 호출될 때 위 클래스가 실행되면 노래가 저절로 재생되었다. 그러나 지금 코드로는 노래를 오로지 처음부터만 재생할 수 있다. 중간부터 재생하려고 재생막대 그 어딘가를 누르게되면, 다시 fetch url을 호출하는 과정이 되기 때문에 다시 노래가 처음부터 시작된다.

**행복하지 않다.**

## 첫번째 시도,

파일을 읽어오는 부분인 `c = f.read()` 부분에 어디서 읽을지 시작부분을 넣어주는 방법이다. `c = f.read(sample_rate * second)` 의 형태이다. 여기서 sample rate란 음악 파일에서 초당 읽어오는 데이터의 양이다. 따라서 시작하고자 하는 위치를 초(second)의 형태로 곱해주면, 파일에서 내가 시작하고자 하는 부분부터 읽어올 수 있기 때문이다.  
sample rate는 음악별로 다르지만, jamendo 홈페이지에서 다운받은 무료 음원의 sample rate은 거의 모두 22050이었다.

### 결과 1

반만 성공함. 프론트엔드에서 지정한 위치를 따로 설정해서 받아오는 것은 작동하지 않았고, second = 70 (초)로 설정해두면, 매번 70초의 위치에서 노래가 시작되기만 했다 ㅋㅋㅋ

## 두번째 시도,

위에 입력한 코드의 `response`에 다른 attribute를 추가해주었다.

1. response['Accept-Ranges'] = 'bytes'
2. response['Content-Length'] = len(open('media/'+music_file, 'rb').read())

```python
class MusicStreamingView(View):
    def get(self, request):
        track_id    = request.GET.get('track_id', None)
        start_sec   = request.GET.get('start_sec', 0)

        if track_id:

            if int(track_id) > len(Track.objects.filter()) or int(track_id) <= 0:
                return JsonResponse({'message' : 'INVALID_KEY'}, status = 400)

            track       = Track.objects.get(id = track_id)
            music_file  = track.music_url
            stream      = self.iteration('media/'+music_file)
            response    = StreamingHttpResponse(stream, status = 200)

            response['Content-Type']        = 'audio/mpeg'
            response['Accept-Ranges']       = 'bytes'
            response['Cache-Control']       = 'no-cache'
            response['Content-Length']      = len(open('media/'+music_file, 'rb').read())
            response['Content-Disposition'] = f'filename = {music_file}'

            return response

        return JsonResponse({'message' : 'INVALID_KEYWORD'}, status = 400)

    def iteration(self, file_name):
        with open(file_name, 'rb+') as f:
           while True:
                content = f.read()
                if content:
                    yield content
                else:
                    break

```

### 결과 2

성공성공~

# 음악 파일 unit test

요즘 개발과 동시에 유닛테스트를 진행하고 있다. 일반 `HttpResponse` 나 `JssonResponse`와는 달리 Streaming music은 어떻게 unit test를 할까?

우선은 지정해둔 내부 속성들이 다 잘 전달되었는지로 확인하기로 했다.

```python
response['Content-Disposition'] = f'filename  = {music_file}'
```

이렇게 추가해두고, test.py 파일은 아래와 같다.

```python

class StreamingTest(TestCase):
     def test_streaming_get_success(self):
         client = Client()
         response = client.get('/music/track?track_id=1')
         self.assertEqual(response.get(
             'Content-Disposition'),
             "filename = DR_GROOVE_GANG_-_A_l_ancienne.mp3"
         )
         self.assertEqual(response.status_code, 200)

```

우선 여기까지로 타협 !

```toc

```
