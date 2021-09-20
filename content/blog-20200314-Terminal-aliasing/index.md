---
emoji: 👑
title: 프로그램 추가 설치 없는 alias로 터미널 살의 질 개선하기
date: '2020-03-14 16:00:00'
author: 이워크
tags: 위코드 wecode 위코드풀스택 wecodefullstack terminal bash zsh
categories: 개발블로그
---

## alias?

> 리눅스의 기본명령어와 긴 명령어를 간단히 줄여서 사용할 수 있게 해주는 쉘내부명령어

즉, 복잡한 명렁어나 여러 옵션을 한 번에 사용하고자 할 때 간단한 이름으로 사용할 수 있도록 하는 명령어다. 중요한 점은 **<span style="color:orange; background-color:beige">쉘 내부 명령어</span>** 라는 것이다. 순수 쉘을 강박적으로 좋아하는 나같은 사람한테 딱이지 ^\_\_\_^

의미는 단축키와 비슷할 수 있지만, 활용도는 단순 단축키 그 이상이다.

# 형식

**alias `별명`=`사용할 명령어`**

이 모양이면 끝난다. 참고해야할 점은 equal 기호 양쪽에 띄어쓰기 노노!

이미 우리 쉘들은 기본적인 alias를 사용하고 있을 수도 있다.

```bash
alias ll="ls -al"
alias cp="cp -i"
```

이정도? 뜻은 **ll** 이란 명령어를 치면, **ls -al** 을 실행시켜라 라는 뜻이 된다.

# 사용 방법

1. .bashrc 또는 .zshrc , 자기가 사용하고 있는 쉘rc 파일을 연다.

2. 파일에서 alias라는 단어를 검색해보자 (명령모드에서 '/alias'를 치고 엔터)

3. 이미 몇 개가 있다면 그 아래에 추가한다.

```bash
alias ll="ls -al"
alias cp="cp -i"
alias myip="hostname -I"
alias pwdc="pwd | xclip"
```

4. 저장하고 빠져나와 `source .bashrc` 혹은 `source .zshrc`로 적용해주기

터미널에서 치는 거의 모든 명령어를 적용할 수 있기 때문에 매우 편리하다. 또 파이프(|)나 세미 콜론을 이용하면 연쇄적인 작업도 할 수 있다.

# 예시 1 이동 줄이기

해야하는 작업이 다음과 같을 때

1. 가상환경 실행
2. 디렉토리로 이동
3. 작업 후 가상환경 비활성화

```bash
alias neck="conda activate django-mysql; cd $HOME/Development/WECODE/devel/project2/necktidal-backend/"
alias de="conda deactivate"
```

매번 터미널을 실행할 때마다 디렉토리로 이동하기 번거로웠는데 **<span style="background-color:pink; padding:2px;">이제는 터미널에 `neck`만 치면 된다.</span>** 작업 끝나고 `de`만 치면 **디엑티베이트**도 할 수 있다.

# 예시 2 실행시키기

AWS 서버 쉽게 실행시키기

```bash
alias aws="ssh -i $HOME/Downloads/key_shlee.pen ubuntu@52.79.109.162"
```

이젠 터미널에서 `aws`만 치면 내 서버 아이피까지 일일이 외우고, key 파일이 있는 디렉토리로 계속 이동하지 않아도 되기 때문에 작업속도가 굉장히 빨라진다.

# 예시 3 단축

외우기 어려운 명령어

`nautilus .`라는 명령어는 현재 내 디렉토리를 GUI로 열어주는 기능이다. 스펠링이 만만치않다. 오타라도 난다면 GUI여는 과정이 너무 짜증날 것이다.

```bash
alias na="nautilus ."
```

# 예시 4 기본 명령어 연쇄

```bash
alias lsd="ls -l | grep '^d'"
```

ls 한 결과물 중에 파일들은 제외하고 디렉토리만 보여줄 때 사용한다. 진짜 쉘 고수들은 이런 연쇄 alias들이 진짜 많다고 한다. 나도 더 편하게 살래 ..

# 내코드

다음은 내가 지금 지정해둔 alias 들이다.

```bash
alias lsd="ls -l | grep '^d'"
alias de="conda deactivate"
alias blog="cd $HOME/Development/blog/"
alias myip="hostname -I"
alias ipc="hostname -I | xclip"
alias pwdc="pwd | xclip"
alias insta="conda activate Instagram_tutorial; cd $HOME/Development/WECODE/devel/instagram/"
alias crl="conda activate scrap01; cd $HOME/Development/WECODE/devel/web_scraping/"
alias dev="cd $HOME/Development/WECODE/devel/"
alias star="conda activate django-mysql; cd $HOME/Development/WECODE/devel/starbucks/"
alias srw="conda activate django-mysql; cd $HOME/Development/WECODE/devel/project1/sariwon/"
alias na="nautilus ."
alias aws="cd $HOME/Downloads/; ssh -i key_shlee.pem ubuntu@52.79.109.162"
alias neck="conda activate django-mysql; cd $HOME/Development/WECODE/devel/project2/necktidal-backend/"
```

# 단점

alias 없이 살 수 없어짐.  
한 일곱 글자 넘어가는 명령어, 세 단계 이상 들어가야하는 경로를 견딜 수 없게 된다...

---

모두 행복한 터미널 생활 !

```toc

```
