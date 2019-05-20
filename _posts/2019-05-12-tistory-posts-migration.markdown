---
layout: post
title: "블로그 소생기 #2"
date: 2019-05-12 09:39:00 +0900
published: false
comments: true
categories: [jekyll]
tags: [jekyll, tistory]
---

## Abstract

블로그를 다시 시작하고 보니 근근이나마 포스팅 해왔던 예전 티스토리의 글들도 함께 옮겨와보고 싶어졌다. 자잘한 지식 조각들이나마 작성했던 글들에 애정이 남아있는데 안타깝게도 티스토리의 게시글 내려받기 API가 서비스가 종료된걸 보고 내용을 로컬에 파일 형태로 보존하고 싶었다.

그리고 사실 100명도 안되는 약소한 숫자이긴 해도 생각보다 꾸준히 방문들 해주셔서 깃헙 페이지로 방문자를 그대로 당겨오고 싶은 마음도 없잖아 있음; 변변찮은 글인데도 꾸준히 클릭들 해주시는 것도 신기하고, 인턴 때 작성했던 ASP 관련 포스트가 순위권에 있는 것도 신기하다.

![tistory-statistic](/images/2019-05-12-tistory-posts-migration-1.png)

## Detail

사실 포스트야 크롤링으로 긁으면 아무 문제 없겠지만, 내용물이 html 형태로 되어있다보니 깃헙 페이지에서 활용하는 마크다운 형식으로, 또 내가 원하는 형태로 변환하는게 쉽지 않다. 마침 [taylor224님의 티스토리 포스트 백업 코드][ref-tistory-backup]를 찾아서 조금 손봐 마크다운 파일로 변환하기로 했다. 자바스크립트와 파이썬 두 가지 버전으로 올려주심. [haxOr님의 깃헙 페이지용 마크다운 파일로 변환하는 코드][ref-tistory-backup-by-ruby]도 찾긴 했는데 일단 루비 기반이라 🙄 향후 집에서 머신러닝 관련 코드 돌려볼 준비도 할 겸 노트북 셋업 후 **taylor224**님의 파이썬 코드를 조금 손봐 사용하기로 했다.

근데 해당 코드로 게시물을 긁을 때 적용하신 TickTalk 스킨이 [2016년 티스토리 개편으로 out-dated 되버림][link-tistory-skin-vault] 😭 대충 긁는 코드랑 이미지 다운로드 하는 코드만 참고하고 html 파싱하는건 알아서 해야겠다. 투덜투덜

적당히 [Anaconda][link-anaconda-download] 다운로드 해주고.

## Reference

- 티스토리 포스트 백업 코드
    
    - To Wordpress by Python: [taylor224님 Gist 페이지][ref-tistory-backup]
    - To Jekyll by Ruby: [haxOr님 블로그 포스트][ref-tistory-backup-by-ruby]

[link-anaconda-download]: https://www.anaconda.com/distribution/#download-section
[link-tistory-skin-vault]: https://est0que.tistory.com/notice/3035
[ref-tistory-backup]: https://gist.github.com/taylor224/5eef306afaef7a7a136c66daecba6e41
[ref-tistory-backup-by-ruby]: https://blog.hax0r.info/2018-02-18/using-jekyll-with-github-pages/
