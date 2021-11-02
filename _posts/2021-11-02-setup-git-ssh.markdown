---
layout: post
title: "Git SSH 설정하기"
date: 2021-11-02 11:29:00 +0900
published: true
comments: true
categories: [git]]
tags: [git, ssh, linux]
---

매번 찾아보기 귀찮으니 Git SSH 설정하는 법을 정리한다. 리눅스 기준으로 정리함.

## SSH Key 생성

아래와 같이 서버에서 SSH Key를 생성 후 `ssh-agent`에 등록한다.

``` sh
# SSH Key 생성
> ssh-keygen
Generating public/private rsa key pair.
# 생성될 키의 경로를 입력
Enter file in which to save the key (/home/aicr/.ssh/id_rsa): id_rsa_test
# SSH Key를 참조 할 때 패스워드 입력 (optional)
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
# SSH Key 생성 결과 출력
Your identification has been saved in id_rsa_test.
Your public key has been saved in id_rsa_test.pub.
The key fingerprint is:
SHA256:449r9b7wluDbdCjeUNNnvehyozjehR6mc1xm/VTJGWE aicr@cl-aicrdev04
The key's randomart image is:
+---[RSA 2048]----+
|             this|
|               is|
|           random|
|              art|
|            image|
+----[SHA256]-----+
```

아래와 같이 `ssh-agent`에 새롭게 생성한 키 중 Private Key를 등록하고

``` sh
# 만약 Could not open a connection to your authentication agent. 에러메시지가 출력된다면:
> eval $(ssh-agent -s)

# SSH Private Key 등록
> ssh-add id_rsa_test
# SSH Key 생성시 passphrase를 등록했다면:
Enter passphrase for id_rsa_test:
Identity added: id_rsa_test (id_rsa_test)
```

아래와 같이 생성된 키 중 Public Key를 복사해둔다.

``` sh
# SSH Public Key 확인
> cat id_rsa_test.pub
ssh-rsa {public-key-string-with-random-characters} {account}@{server}
```

## Git SSH Key 등록

사내깃이 올드한게 또 스트레스네; 프로세스는 비슷할 것이라 생각함.

공식 페이지 내 프로필의 드롭다운 메뉴에서:

1. Settings 항목 선택
2. SSH and GPG keys 항목 선택
3. New SSH Key 버튼 클릭
4. 식별을 위한 Title을 입력하고 Key에 생성한 SSH Public Key의 내용을 입력 후 Add SSH Key 클릭
