---
layout: post
title:  "블로그 테마 변경 - Jekyll-Uno"
date:   2017-06-25 20:00:00 +0900
# comments: true
categories: jekyll
tags: [jekyll]
---
첫 블로그 개설 후 약 2주를 방치해두다가 [함수형 자바스크립트 프로그래밍](https://kitchu0401.github.io/2017/js-fnc-programming/) 세미나 후기를 계기로 글을 적는다. [Taken](https://github.com/vfalanis/taken) 테마로 `jekyll` 스타트를 끊었지만 최초 설정에 이것저것 애도 먹고 커밋 전에는 로컬에서 저장한 글이 `_site` 폴더에 배포가 되지 않는 문제 - *물론 이건 내 셋업에 문제가 있었을 수도 있다* - 가 있어 마음에 썩 들지 않았다. 블로그 방치했던 지난 2주와 이런 부분이 아예 관계가 없진 않겠지.

어쨌든, 마뜩찮은 상태의 블로그에 글에 적고싶은 기분이 영 나질 않아 갤러리를 둘러봤더니 마침 깔끔한 테마가 눈에 띄길래 냉큼 집어왔다. [Jekyll-Uno](https://github.com/joshgerdes/jekyll-uno) 테마다. 메인 페이지는 간결하게, 포스트 상세 페이지도 데스크탑 기준으로 메인 페이지를 사이드로 밀어놓고 우측 절반 정도만 할당하는 심플한 테마다. 다른 분들의 블로그도 둘러보고 내 스스로도 테마를 올려보면서 느낀건 개인적으로 포스트 본문의 좌우길이가 좁은 편이 취향이라는 것. 코드를 작성 할 때 좌우 폭이 아쉬울수도 있겠지만 그건 차후 별도의 스타일이나 플러그인으로 해결해야겠다. 추가로, 이번 테마는 **로컬에서 서버를 띄울 때에도 정상적으로 글이 배포**된다! **HAYO**!

이번에도 설치에 자잘한 문제들이 있었지만 역시 구글에서 운 좋게 해답을 찾을 수 있었다. `bundler` 설치가 요구되지 않았던 **Taken** 테마와 달리 (결국 오류가 발생해서 별도로 설치는 해줬지만), **Uno** 테마는 명시적으로 `ruby` 및 `rubygems` 의 설치 필요성을 언급하고 있다. 맥을 사용중이라 이건 사실 큰 문제가 안됐는데.. 문제는 `bundle install` 커맨드로 의존성 셋업을 수행하는 과정에서 다음과 같은 오류가 발생했다.

``` sh
libxml2 is missing.  Please locate mkmf.log to investigate how it is failing.
```

사실 저렇게 한 줄만 얘기해줬다면 문제가 정확히 뭔지 알았을텐데 (라고 변명해봅니다) 초기화 과정에서 출력되는 로그들과 함께 해당 오류 문구가 섞여있어 처음엔 잠시 헤맸다. `libxml2` 플러그인의 설치에 문제가 발생한 것으로, [stackoverflow.com](https://stackoverflow.com/questions/23668684/nokogiri-failed-to-build-gem-native-extension-when-i-run-bundle-install)에서 확인 할 수 있는 답변대로 해당 라이브러리를 수동으로 설치 후 설치된 라이브러리를 참조하도록 다음 명령을 수행해준다. 내 경우는 이후 큰 문제 없이 셋업 완료되어 `localhost:4000/jekyll-uno/` 에서 배포 결과를 확인 할 수 있었다.

``` sh
brew install libxml2
bundle config build.nokogiri "--use-system-libraries --with-xml2-include=/usr/local/opt/libxml2/include/libxml2"
bundle install
```

![changing-blog-theme-1](/images/2017-06-25-changing-blog-theme-1.png)

사소하게도 포스트 상세 페이지에서 딱 봐도 뭔가 syntax 관련된 것처럼 보이는 현상이 발생하길래 파일을 좀 뒤져봤더니 아예 **vscode editoer**에서 제대로 파싱이 안되길래 `_includes/head.html` 파일의 `page.excerpt | remove: '<p>' | remove: '</p>'` 코드가 들어간 모든 행을 주석처리하니 문제가 사라졌다. - **마음대로 안되면 부순다** -

맘에 안드는 스타일 조금 수정하고 `_config.yml` 조금 수정하고.. 손을 더 대다간 일이 어디까지 커질지 모르니 이대로 쓰도록 한다.ㅎㅎ 세미나 후기 포스팅을 시작으로 꾸준히 가보자꾸나. 기본 설정된 Syntax Highlighter 스타일 상당히 마음에 안드는데 언젠가 또 손을 대고싶어지는 날이 오겠지!╭( ･ㅂ･)و
