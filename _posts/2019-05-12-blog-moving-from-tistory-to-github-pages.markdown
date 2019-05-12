---
layout: post
title: "블로그 소생기 #1"
date: 2019-05-12 04:30:00 +0900
comments: true
categories: [jekyll]
tags: [jekyll, disqus]
---

## Abstract

오래간만에 블로그 저장소를 살펴보니 [티스토리](kitchu.tistory.com)에서 옮겨와 몇 건 되지 않는 포스팅을 적다 멈춘게 2017년 여름 경이다. 당시 블로그에 포스팅 할 때마다 푸시 이력에 반영되는 것도 싫고, 소스 관리 한다고 브랜치를 둘로 나누어 관리하다가 css파일이 깨졌는지 스타일이 망가져버렸었다. 쿠크다스 멘탈인 나는 고쳐보겠다고 붙잡고 있다가 쥐쥐쳐버림 (ಥ_ಥ)

## Basic

[jekyll](https://jekyllrb.com/)도 그 사이 메이저 버전업이 됐고, 스킨 유지하자고 망가진 코드 복구하느니 마침 윈도우 포맷해서 밀어버린 김에 쿨하게 백업한 다음 저장소 날려버리고 새로 만든다.

이전과 마찬가지로, [Ruby를 먼저 설치](https://www.ruby-lang.org/ko/downloads/)한다. 아래의 명령어를 실행해 `jekyll`과 `bundler`를 함께 설치해준다.

```
gem install jekyll bundler
```

이후 아래의 명령어를 실행하면 현재 폴더에 `{PROJECT_NAME}` 폴더가 생성된다.

``` 
jekyll new {PROJECT_NAME}
```

생성된 폴더에서 아래의 명령어를 실행하면 `http://localhost:4000` 경로에 접근 가능한 웹서버가 뜨고, 실제로 반영되었을 때 보여지는 모습을 확인 할 수 있다. `--watch` 파라미터를 추가하면 `hot-deploy` 형태로 물리적으로 파일이 수정 될 때마다 소스가 반영되는걸 확인 할 수 있음.

```
bundler exec jekyll serve --watch
```

아래 양식의 샘플 포스트가 하나 생성되는데, 동일한 양식으로 포스트를 작성하고 깃헙 페이지를 적용할 저장소에 푸시하면 수 분 내로 반영된 포스트를 확인 할 수 있다. `.gitignore` 파일에 명시된대로, `jekyll serve` 또는 `build` 커맨드를 실행해 생성되는 `_site` 폴더 아래의 산출물들은 저장소에 푸시 할 필요 없고 템플릿 그대로 푸시하면 깃헙 페이지 내에서 동적으로 배포하는듯.

``` md
---
layout: post
title:  "Welcome to jekyll!"
date:   2019-05-11 19:08:58 +0900
categories: jekyll update
---

*포스팅 내용*
```

## Disqus 연동

`jekyll` 기본 템플릿에서는 About 페이지와 포스트 목록 및 상세조회 외에는 다른 기능이 지원되지 않는다. 플랫폼에 독립적으로 댓글 작성 기능을 제공하는 `Disqus` 연동 관련하여 자세하게 작성된 [포스트][ref-disqus]가 있지만, `jekyll new` 커맨드로 생성한 폴더에는 `_layout` 폴더가 존재하지 않는다.

[jekyll 공식문서][ref-jekyll-layout]를 참고하여 위의 명령어를 실행하면 `jekyll` 기본 테마인 `minima`가 설치된 경로를 확인 할 수 있으며, `_includes`, `_layouts`, `_sass` 등의 폴더를 확인 할 수 있다.

```
bundle show minima
# C:\Ruby25-x64\lib\ruby\gems\2.5.0\gems\minima-2.5.0
```

저장소 폴더에 `_layouts` 폴더를 생성하고 동일한 이름의 `post.html` 레이아웃 파일을 생성하면 공용 참조 폴더가 아닌 로컬의 파일 내용을 먼저 참조하게 된다. [포스트][ref-disqus]에서 알려주는대로 `disqus` 관련 코드를 넣어주려고 하는데.. `minima` 테마는 기본적으로 변수가 지정되어 있으면 아래와 같이 `disqus` 코드를 출력하도록 되어있다.-_-

#### `_layout/post.html`

![disqus-screenshot-1](/images/2019-05-12-blog-moving-from-tistory-to-github-pages-1.png)

- `if site.disqus.shortname != blank`의 원래 코드는 `if site.disqus.shortname`였다. true or false를 체크하는 코드로만 동작하는지 제대로 작동하지 않아 `!= blank`로 변경하니 비로소 하위 코드를 탄다.
- `if page.comments`는 `minima` 테마의 `post.html` 파일에는 존재하지 않는 코드.
- `include disqus_comments.html` 코드에서 `include`는 `_includes` 폴더 안의 템플릿 조각을 참조한다. `_layout`에 생성하고 안된다고 삽질하지 말자. 내가 그랬다. (ಥ_ಥ)

#### `_config.yml`

![disqus-screenshot-2](/images/2019-05-12-blog-moving-from-tistory-to-github-pages-2.png)

- [jekyll 공식문서][ref-jekyll-variables]를 참고하면 `site` 변수는 `_config.yml` 파일에 정의하면 페이지에서 참고 할 수 있다고 한다. 해당 파일에 위와 같이 원하는 변수명으로 변수를 추가 한 뒤 해당 변수를 `post.html`에서 참고하도록 코드를 작성하면 된다.

#### `_includes/disqus_comments.html`

![disqus-screenshot-3](/images/2019-05-12-blog-moving-from-tistory-to-github-pages-3.png)

- `var disqus_config = function () { .. }` 부분은 `disqus` 홈페이지에서 예제 코드를 긁어오면 주석처리가 되어있다. `this.page.url` 및 `this.page.id`를 각각 적당한 문자열로 대치해준 뒤 주석을 해제해주자.
- 각각의 변수는 도메인 / 게시글 별로 유일 식별을 위해 사용되는 듯 하다. 다만 `this.page.url`의 경우 대충 적어넣으면 유효한 url 형태의 값을 넣으라고 콘솔 에러가 발생하니 실제 댓글이 표시될 페이지의 url을 알맞게 넣어주자.
- `page.id` 및 `page.id`는 각 포스트 내에서 참조 가능한 변수다. [jekyll 공식문서][ref-jekyll-variables]를 참고 할 것.

여기까지 처리하면 댓글 연동이 정상적으로 작동한다. 아무것도 나타나지 않는다면 소스 코드에서 분기문을 작성해 하위 코드를 제대로 타는지 체크하고, 브라우저에서 개발자 콘솔을 열어 혹시 오류 메시지가 나타나는지 확인해보자. 그림 링크가 아니라 코드로 적고 싶었는데 `liquid` 문법으로 적으면 페이지 배포시 컴파일이 되버려서 출력이 안됨. 깃헙은 역시 사진 첨부는 아주 불편하고 ㅂㄷㅂㄷ.. 컴파일 안되게 처리하는 방법이 있겠지만 시간 소모 할 필요 없는건 그냥 대충 넘어간다.

포스트 제목에 넘버링을 한 이유는:

1. 태그 별 모아보는 목록을 따로 만들어보고
2. 티스토리에 작성해둔 포스트들을 옮겨와보고 싶어서다.

일종의 TODO 목록으로 남겨둠.

## 참고

- Disqus 관련: [야쿠자님 깃헙 페이지][ref-disqus]

[ref-jekyll-layout]: https://jekyllrb-ko.github.io/docs/structure/
[ref-jekyll-variables]: https://jekyllrb-ko.github.io/docs/variables/
[ref-disqus]: https://dev-yakuza.github.io/ko/jekyll/disqus/
