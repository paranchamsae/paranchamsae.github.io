---
layout: post
title: 🪶 git pull, fetch, clone의 차이
sitemap: false
hide_last_modified: true
---

매번 다른 PC에서 작업하시는 분들, 또는 신규 팀원이 프로젝트에 참여하게 되었을 때, 기타 등등!<br/>
프로젝트가 Git으로 형상관리 중이라면 remote repository에서 코드를 가져 와야겠죠?<br/><br/>

그런데 <span style="color: lightgray;">~~아직도~~</span> clone이나 pull이나 remote repository에서 코드 가져오는 것 아닌가?<br/>
않이 근데... pull도 받아오는 의미고 fetch도 받아오는 의미잖아.. 할 때마다 헷갈리네 ㅠ_ㅠ 라고 생각하는 저(?)를 위해!

이번 글에서는 <span style="background-color:#fff5b1">**git pull, fetch, clone의 차이**</span>에 대해 정리 해 볼게요!

<!-- * toc -->
<!-- {:toc} -->

## 📖 알아보...기 전에!

아래로 내려가기 전 짧게! git 명령어 몇 개만 짚고 가볼게요.

태초에(?) 아무것도 연동되지 않은 날것의 폴더에 프로젝트를 생성하고, 형상관리 하고 싶어요! 라고 했을 때, 처음 사용하는 명령어는?
```console
$ git init
```
이었죠? 형상관리를 위한 <span style="background-color: #fff5b1">local repository를 처음 초기화</span> 할 때 사용하는 명령어입니다.<br/><br/>

두 번째! <span style="background-color: #fff5b1">local repository를 remote repository와 연결짓기 위한</span> 명령어, 또는 <span style="background-color: #fff5b1">현재 local repository에 연결된 remote repository 정보를 알고싶을 때</span> 사용하는 명령어는?
```console
$ git remote
$ git remote -v
```
였어요. 우리는 remote 명령을 사용해 나의 수정 내역을 반영할 remote repository의 주소를 설정도 할 수 있고, 현재 local repository에 설정된 remote repository의 주소도 알 수 있었습니다.<br/><br/>

세 번째! 형상관리 중에 작업을 분리하기 위해 서로 다른 작업 브랜치를 만들었다고 쳐요.<br/>
그러다가 브랜치에서 자잘한 작업을 마무리 짓고 작업 내역을 브랜치로 통합 하고 싶다! 했을 때 사용하는 명령어는?
```console
$ git merge
```
였습니다. <span style="color: lightgray;">( git merge와 conflict 관련된 내용도 나중에 좀 더 자세히 다뤄볼게요 😇 )</span>


위 세 개의 명령어를 배경지식으로 깔고, 본격적으로 pull, fetch, clone의 차이에 대해 살펴보아요 :)

## 📖 git pull

pull의 사전적 정의는 끌다, 당기다의 의미를 가지고 있죠? Git에서의 pull도 착실하게(?) 그 역할을 수행 합니다.<br/>
Local repository에서 pull을 사용하면, <span style="background-color: #fff5b1">**연결된 remote repository**</span>의 최신 내용의 코드를 내 local repository로 가져와, <span style="background-color: #fff5b1">**merge**</span> 처리까지 해 줍니다.

자 그러니까 이해하기 쉽게, 위 문장을 하이라이팅 된 곳만 살펴 보면,<br/>
  1. 연결된 remote repository<br/>
    - 여기서 **연결된** 에 주목! 이미 내 local repository는 remote repository와 연결이 되어 있대요.<br/>
    그러면 이미 local repository에서 git init과 git remote 설정을 모두 거친 상태겠네요?
  2. merge<br/>
    - merge는  서로 다른 브랜치를 하나로 병합 해 주는 명령어다! 라는 내용을 위에서 확인하고 왔죠?

결론적으로 git pull을 사용했을 때 얻을 수 있는 효과는<br/>
> remote repository의 가장 최근 작업 내역을 local repository로 가져옴과 동시에 덮어 쓴다.

라고 정리해도 되겠네요 :)

## 📖 git fetch

fetch도 마찬가지로 어딘가에서 뭔가를 가져온다- 라는 뜻이잖아요?<br/>
pull과 마찬가지로 fetch도 remote repository에서 정보를 가져올 때 사용하는 명령어입니다.

단, fetch는 remote repository의 내용을 확인하기는 하는데, <span style="background-color: #fff5b1">**local repository와 merge는 해 주지 않아요.**</span><br/><br/>그러니까, fetch는
> local repository보다 remote repository의 코드가 더 최신이라 하더라도,<br/> 이 코드 내용을 local repository와 동기화 시켜주지는 않는다.

라고 한 줄 정리 되었네요 :)

## 📖 git clone

clone은 어디에선가 뭔가를 복제해온다- 라는 느낌이 강하게 오죠?<br/>
clone은 remote repository에서 코드를 로컬로 가져오기 위해 사용하는 명령어로, pull과 비슷한 일을 합니다.

그런데 pull과 다르게 clone은 <span style="background-color: #fff5b1">local repository에 대한 세팅이 되어있지 않더라도 사용</span>할 수 있습니다. 이게 뭔 말이냐-<br/><br/>
우리는 위에서 pull에 대해 살펴보면서 하이라이팅 처리가 되어있던 <u>연결된 remote repository</u> 라는 문장을 읽었잖아요?<br/>
그래서 <u>pull을 사용하기 전에 git remote를 사용해서 연결된 remote repository가 있는지를 먼저 봐야 하겠구나-</u><br/> 라는 사실을 확인하고 왔어요. <br/>그런데? clone에는 그런 말이 없습니다.

여기에 pull과 clone의 차이점이 있습니다.

clone을 사용하면 내 local repository에 git과 관련된 초기화 작업이 되어있지 않더라도,<br/>
즉시 <span style="background-color: #fff5b1">remote repository와 로컬을 상호 연결시킴</span>과 동시에 <span style="background-color: #fff5b1">현재 remote repository의 가장 최신 코드를 로컬로 받아</span> 옵니다.

그럼 clone도 한 문장으로 정리 해 볼까요?
> git clone = git init + git remote + git pull

## 💁🏻 결론!

* 어딘가에서 누군가가 커밋한 내용을 가져올 땐 pull을 사용 해 보아요.
* pull 하기 전에 fetch 먼저 해 보는 습관을 들여보도록 해 보아요. 무턱대고 pull 했다가 내 로컬의 피 땀 눈물이 그대로 증발하는<br/>대참사가😇 날 수 있어요.
* 처음 프로젝트에 투입된 새 직원 분을 위해서는 git remote repository 주소를 주시면서 "clone 해 놓으세요~" 라고 말씀 드려 보세요. <span style="color: lightgray">~~친절함에 감동하실겁니다.~~</span>

<br/>
여기까지 git pull, fetch, clone의 차이에 대해 정리 해 보았습니다.<br/><br/>
그럼, 다음 글에서 또 뵈어요 :) 🪶