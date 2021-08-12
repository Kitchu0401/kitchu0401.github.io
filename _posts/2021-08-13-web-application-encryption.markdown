---
layout: post
title: "웹 어플리케이션 평문 암호화"
date: 2021-08-13 00:25:00 +0900
published: true
comments: true
categories: [web-development]
tags: [web, web-application, encrypt, encryption,js, javascript, node-js, python, flask]
---

개발 중인 과제에서 아이디 및 비밀번호 암호화가 요구되어 기록으로 남길 겸 정리해둔다. 사실 기록이라고 해봤자 요구조건 정도만 충족하고 필요한 만큼만 보충 할 것임. 사실 어플리케이션에서 패스워드 등의 암호화는 기본이라고 생각하지만 딥하게 파보기엔 머리도 아프고 할 일은 쌓여있는데 시간은 없다..

## Simplest

가장 간단하게 "일단 평문만 아니게만 만들어보자" 컨셉으로 간다.

작성자가 개발 진행중인 어플리케이션은 프론트엔드는 VueJS로, 백엔드는 Flask로 되어있다. 이 말은 곧, 데이터를 송신하는 쪽은 Javascript이며 수신하는 쪽은 Python이란 이야기다.

사실 이번 과제를 개발하며 암복호화에 대해 처음 고민해보게 되었는데, '비밀번호의 암복호화' 라고 말하면 당연한 내용이지만 이를 어플리케이션, 즉 앱 - 한 글자로만 가자 요즘 손목이 조금씩 아프다 ㅜㅠ - 에 적용하려면 앱의 프로세스에 녹여낼 수 있는 암복호화 방식(단방향 / 양방향), 알고리즘의 선택, 직접 구현하는 수준이 아니며 작성자의 케이스처럼 이종 언어 이슈가 있다면 구현체까지 동일한 결과를 내놓는 라이브러리를 찾아야한다. 사실 *구현체까지 동일한 결과를 내놓는 라이브러리를 찾아야한다* 에 시간을 많이 투자하고싶지 않아 `base64` 인코딩으로 평문 상태만 면하고 요건 통과를 기도하는 방향으로 가닥을 잡기로 했다.

VueJS 백 단의 NodeJS 내장 모듈 중 `Buffer`가, 파이썬은 `base64` 모듈이 각각 원하는 기능을 제공한다. 자세한건 코드로 대신한다.

``` js
const { Buffer } = require('buffer')

// 인코딩
const ENCODING = 'utf-8'

// 데이터
const USER_ID = 'kitchu'
const PASSWORD = 'this-is-password'

const encode = (rawString) => Buffer.from(rawString, ENCODING).toString('base64')

console.log( encode(USER_ID) )  // a210Y2h1
console.log( encode(PASSWORD) )  // dGhpcy1pcy1wYXNzd29yZA==

// 오브젝트형 인코딩은 JSON을 활용한다.
// 인코딩 결과는 적지 않는다. 복사가 안되서 한글자씩 쳐야하는데 ㅈㄴ 길다 ㅜ
console.log( encode(JSON.stringify({'userId': USER_ID, 'password': PASSWORD})) )

// Buffer 디코딩은 귀찮으니 생략
```

``` py
import base64
import json

# 인코딩
ENCODING = 'utf-8'

# base64.b64decode 함수가 바이트 배열을 인자로 받음에 주의
decode = lambda e: base64.b64decode(bytes(e, 'utf-8')).decode('utf-8')

print( decode('a210Y2h1') )  # kitchu
print( decode('dGhpcy1pcy1wYXNzd29yZA') )  # this-is-password

# 오브젝트형 인코딩은 역시 JSON 활용
print( json.loads(decode('...')) )  # 위에꺼 넣어..
```

## (일단) 맺음말

사실 이렇게 단순 인코딩으로 평문만 면하는 조치는 포스팅하는 입장에서 할 말은 아니지만; 별다른 암복호화 키가 존재하지 않아도 원문을 바로 까볼 수 있으니 암복호화라고 할 수도 없는 수준이다. 상용 수준의, 외부 사용자들에게 공개되는 어플리케이션에서는 절대 적용하지 말도록 하자. 요건 불충족시 후속 시리즈로 돌아옴.
