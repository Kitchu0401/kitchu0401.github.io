---
layout: post
title:  "[세미나 후기] 오픈소스 개발자 이야기"
date:   2017-07-02 20:00:00 +0900
comments: true
categories: review
tags: [review, seminar, oss, open-source, open-source-software, code-review]
---
서너 달 전부터 활발하게 토이 프로젝트를 진행해왔지만, 지극히 흥미 본위의 도메인 & 일단 나부터 쓰고보자[^1]는 식의 코드 작성으로 인해 깃헙에 올려뒀으면서도 누군가의 PR은 상상도 하지 못했다. 개발문화가 잘 잡혀있는 기업에 근무중인 지인으로부터 운 좋게도 조언을 받아 브랜칭과 이슈 / Pull requesting을 직접 해볼 수 있었고[^2], 이러한 개발자 간의 소통에 맛을 들인게 최근이었던지라 [오픈소스 개발자 이야기](http://onoffmix.com/event/102936) 소식을 확인 했을 때 기대를 안고 참여신청을 했다. 페이스북 [OSS 개발자 포럼](https://www.facebook.com/groups/ossdevforum/)과 국민대학교에서 주관하는 이 세미나는 국내외의 오픈소스에 풍부하게 기여해온 여섯 분 선배님들의 관련된 각양각색 생생한 경험담을 들을 수 있었던 즐거운 자리였다.

각 발표자분들의 이야기와 느낀 점들을 간략히 요약해본다.

1. 오픈소스 개발자의 공부 이야기 by [강대명](https://github.com/charsyam) 님

    [강연 슬라이드](https://www.slideshare.net/charsyam2/how-to-study-77423848)

    전반적으로 (오픈소스) 개발자라면 무엇을 어떻게 공부해야 하는지에 대해 러프한 포인트를 짚어주시는 느낌. 1만 시간의 법칙을 인용, 두어 가지 사례와 함께 기계적인 공부보다 의식적인(집중적인) 공부의 중요성을 오프너로 강조하시고, 이어 피드백과 교정의 중요성을 차례로 강조해주셨다. 개인적으로 새로운 언어는 기존에 만들었던 소스를 배우고자 하는 언어로 다시 만들어보고, 새로운 기술은 토이 프로젝트를 진행하며 습득하신다고.

    다양한 분야를 커버하신 커리어와 함께 나이의 많고 적음, 지위의 고하를 막론하고 적극적으로 질문하신다는 대목이 인상깊었으며, 다음의 두 인용구도 좋았다. 기술은 문제 해결을 위한 도구에 불과함에 잊지 말고, 막힘과 실패를 두려워하지 말도록 하자.

    > Technology is the answer, but what was the question?

    > 적어도 실패는 시작하지 않는 것보다 훨씬 더 큰 결과를 남긴다.

2. 나는 오픈소스로 화가가 됐다 by [김양원](https://github.com/rhiokim) 님

    [강연 슬라이드](https://www.slideshare.net/rhio.kim/open-source-ii-77434894)

    블로그 엔진 [Haroopress](https://github.com/rhiokim/haroopress)와 마크다운 에디터 [Haroopad](https://github.com/rhiokim/haroopad) 프로젝트 오너. 사실 두 어플리케이션을 잘 몰랐지만 깃헙 Star 카운트만 봐도 인지도를 짐작 할 수 있었고, 강연 내내 프로젝트에 대한 발표자 분의 열정과 자부심을 느낄 수 있었다. (심지어 Haroopad는 다국어화가 한국어와 영어 포함 무려 14개 언어로 되어있다;)

    프로젝트의 실제 개발 파트 뿐 아니라 문서화와 홍보에 관련된 생생하고 자세한 경험담을 들을 수 있었으며, 2년 4개월 동안의 롱-런한 개발 / 유지보수를 이어나갈 수 있었던 원동력에 대한 인사이트를 짚을 수 있었던 시간이었다. 역시 어플리케이션의 생명력은 사용자. 그리고 오픈소스 개발자는 배가 고프다..ㅜㅜㅜㅜ (´• ω •̥`)

    ![What is the best editor?](https://lh5.googleusercontent.com/-uC_BOp6TawA/UxSc5fhncWI/AAAAAAAABR0/SalxkmcUCug/w506-h750/developer_humour.png)

    여담으로 Haroopad의 홍보 과정에서 [일본 커뮤니티](http://moongift.jp/)의 케이스와 비교했을 때 오픈소스를 발견하고 전파하는 국내 문화에 대해 약간 아쉬운 감정을 비치셨는데, 이 부분은 일전 *'국내 사용자들은 아직 Starring에 비교적 인색하거나, 최소한 적극적이지는 않은 것 같다.'* 는 지인의 이야기가 생각나 어느정도 공감을 할 수 있었다. 나 본인만 해도 긍정적인 평가에 인색할 필요는 없다고 생각하지만, 프로젝트 오너의 업적에 대해 칭찬의 의미보다는 후일 프로젝트 코드를 참고할 일이 생길지 모르는 경우를 생각해 레퍼런스의 개념으로 Starring을 생각하는 경향이 어느정도 있으니.. 

3. 나의 오픈소스 이야기 by [김준기](https://github.com/achimnol) 님

    [강연 슬라이드](https://speakerdeck.com/achimnol/oss-gaebalja-poreom-nayi-opeunsoseu-iyagi)

    - 다음 티스토리의 모체가 되는 [Tattertools](https://ko.wikipedia.org/wiki/%ED%83%9C%ED%84%B0%ED%88%B4%EC%A6%88) ([Textcube](https://github.com/Needlworks/Textcube/)로 변경)
    - PuTTY 한글화 프로젝트 [iPuTTY](https://github.com/iPuTTY/iPuTTY)
    - 고성능 패킷 프로세싱 프레임워크 [NBA](https://github.com/ANLAB-KAIST/NBA)
    - Lablup의 프로덕트로써 쉽고 빠른 머신러닝 코드 실행을 위한 [Sorna](https://github.com/lablup/sorna)
    - 비동기 파이썬 도커 API [aiodocker](https://github.com/aio-libs/aiodocker)
    - 파이썬 asyncio 모듈 유틸리티 [aiotools](https://github.com/achimnol/aiotools)

    [Lablup](http://www.lablup.com/) CTO, 다채로운 오픈소스 개발이력을 보여주셨던 분.
    
    다양한 오픈소스를 거치며 느껴졌던 성장 포인트를 프로젝트 각각의 스토리와 함께 공유해주셨다. 개인적으로는 완연하게 익숙한 블로그 서비스인 티스토리의 기반이 되는 프로젝트의 실제 개발을 담당하셨던 부분과 함께, 프로젝트에 대한 전권이 상용 소프트웨어와 비교했을 때 매우 자유롭게 이전되는 스토리들도 흥미로웠다. 다가오는 PyCon 2017에서 [aiotools를 소개하는 자리](https://www.pycon.kr/2017/program/184)를 가지신다니 비록 파알못(..)이지만 파이콘을 기다릴 이유 하나가 늘어난 것이렸다.
    
    오픈소스 생태계 진입과 관련한 기본 소양 - 적극성 / 영어 / Git 관련 기술 습득 - 에 더하여 코드리뷰 피드백에 대한 태도를 강조해주셨는데, 사실 이 부분은 강연자 분들 모두 강조하셨던 부분으로 코드리뷰의 대상은 코드일 뿐, 나 자신이 아님을 항상 명심하라는 것.

4. Microsoft 오픈소스의 종류와 활용법 by [김영욱](https://www.facebook.com/winkey7) 님

    마이크로소프트 Technical [Evangelist](http://goodhyun.com/692)로 계신 김영욱 님의 강연이다. 강연 뿐 아니라 세미나 전반에 걸쳐 즐겁게 분위기를 주도해주셨는데, 강연에서는 몇 가지 재미있는 에피소드와 함께 마이크로소프트의 오픈소스에 대한 정책과 마이크로소프트에서 제공하는 Microsoft Azure Service에 대한 소개를 들을 수 있었다. 강연 중 언급해주셨던 [자마린](https://msdn.microsoft.com/ko-kr/library/mt299001.aspx)을 활용한 [앱 개발 추천/후기 글](http://cafe.naver.com/mcbugi/333708)이 있어 링크해둠.
    
    마이크로소프트가 - 기업 규모와 유연함의 상관관계에 대한 개인적인 편견을 물론 감안했을 때 - 개발문화가 평균 이상으로 유연 할 것이라 기대했던건 아니지만, 김영욱 님이 개인적으로 파이썬 공부를 시작하실 때의 사내 반응이 조금 격했다 싶어 의외였다. 이 부분은 강연자분이 재미를 위해 과장하신 부분일지도.ㅎㅎ

5. 우아한 오픈소스 이야기 by [이준호](https://github.com/shallaa) 님

    [강연 슬라이드](https://www.slideshare.net/shallaa/ss-77433342)

    우아한형제들(재미있게 봤던 [기술블로그](http://woowabros.github.io/) - 글 리젠이 뜸해진 것 같지만 (´•ω•̥\`))에서 1년째 근무중이신 분. 세미나에서 강연해주신 여섯 발표자 분들의 어조가 대체로 밝은 분위기였다면, 이준호 님은 - 조금 긴장하셨는지 - 약간 건조한 분위기로 강연을 진행해주셨는데, 오픈소스와 코드리뷰, 스타트업의 비교적 실질적이고 객관적인 면면이 담긴 강연내용에 설득력이 더해지는 느낌이었다.
    
    오픈소스 유지관리에서의 중요한 부분, 코드리뷰와 스타트업에 대한 경험담과 함께, `스타트업에서 오픈소스 하는 개발자` 섹션에서 우아한형제들에서 공개한 [WoowahanJS 프레임워크](https://github.com/woowabros/WoowahanJS)에 대해 이야기하시며 프론트엔드 프레임워크, 나아가 오픈소스화 하는 과정에서의 중요했던 포인트를 잘 짚어주셨던 것 같아 좋았다. 혼자 일하는 환경이 아닌, 협업하는 환경이 기본인 실무에서 프레임워크가 장밋빛 미래만 보장해주는가? 오픈소스화 하는 과정에서 개인과 회사가 각각 얻을 수 있는 결과들은 무엇일까? 등의 질문들도 실제로 겪어보지 않으면 답을 생각해보기 쉽지 않은 부분이라는 생각이 들었던 강연이었다.

6. 오픈소스와 코드리뷰 by [서주영](https://github.com/seoz) 님

    [강연 슬라이드](https://www.slideshare.net/seojuyung/ss-77423315)

    구글 유튜브에서 근무중이신 서주영 님. 규모가 3-40명 가량 되는 규모의 스타트업에서 코드리뷰가 일체 없는 것을 보고 코드리뷰 문화에 대해 전파해야겠다고 결심하셨다고 한다. 코드리뷰의 장점을 다음과 같이 강조해주셨다.

    - **개발자로서의 실력 향상**
        
        실력 향상의 측면에서:

        - 공짜로 받는 코딩 과외
        - 오픈소스에 관한 전문가의 리뷰를 받을 수 있음
        - 고로, 해당 오픈소스 더 잘 이해 할 수 있음
        - 관련 지식을 빠르고 정확하게 학습
        - 올바른 방향에 대한 가이드를 받음 - 코드에 앞서 방향성이 중요 할 수 있다.

    - 커뮤니케이션 능력 향상
    - 영어 공부
    - 뛰어난 개발자들과 연결
    - 자극, 열정
    - 경력, 커리어패스
    - 명예

    이러한 코드리뷰가 가지는 장점들을 서주영 님 본인이 거쳐왔던 다양한 오픈소스 기여 경험 - 메인테이너 / 컨트리뷰터 양 측 모두의 입장에서 - 에 비추어 설명해주셨다. 더불어 삼성전자에 Master Engineer로 재직 중인 Carsten Haitlzer씨의 코드리뷰에 대한 즉석 인터뷰를 공유해주셨는데 센스있는 자막으로 모두에게 큰 웃음을 주셨다.

    <p>
      <figure class="video_container">
        <iframe width="560" height="315" src="https://www.youtube.com/embed/6OehY7D5Ex0" frameborder="0" allowfullscreen></iframe>
      </figure>
    </p>

    요약하자면 코드리뷰는:

    - 내 코드가 잘못되었음을 알려주고 무엇이 허용되고 허용되지 않는지 알려준다.
    - 코드리뷰는 새로운 입문자가 프로젝트에 입문하는걸 도와준다.
    - 사람들이 빨리, 많이 배울 수 있게 해준다.
    - 리뷰에 대한 피드백이 때로 가혹해서 빡칠 때도 있지만, 솔직하기 때문이다.
    - 항상 코드에 대한 것이지 너에 대한 평가가 아니라는 것을 명심해야 한다.

    마지막으로 오픈소스에 기여하면서 지켜야 할 부분들을 짚어주셨는데 요지는 적극적인 자세와 함께 자신의 코드에 책임감을 가지는 것. 다수의 사람이 쓴 코드여도 한 사람이 쓴 것처럼 보여야한다는 말은 오픈소스에서만 지켜야 할 부분은 아니지만, 오픈소스이기에 꼭 필요한 부분이라는 생각이 들었다.

오픈소스라는 단어가 개발자에게 일상적인 단어가 된지 많은 시간이 흘렀다. 연차가 부족해서일수도, 딱 오픈소스를 가져다 써서 문제를 해결하는 데까지만 관심이 있어서일수도, 외국어라는 이질적인 장벽이 가로막고 있어서인지는 모르겠지만, 가져다 쓰는데에는 어려움이 없어도 능동적으로 프로젝트에 PR을 보내고 자신의 의견을 어필하는 것은 아직 먼 나라 이야기처럼 느껴지는게 현실이다.

그래도 이번 세미나를 들으며 멀게만 느껴졌던 오픈소스 생태계가 마치 옆집 형이 들려주는 얘기처럼 친근하게 다가왔다는 사실 하나만으로 내 안의 간극이 좁혀지는 느낌이 들었다. 무엇보다 열정적인 발표자 분들의 태도 - 질문과 답변 섹션에 김영욱 에반젤리스트 님이 말씀해주셨다. 이 분들에게 이건 **일 뿐인게 아니라 일종의 놀이**일거라고. :) - 에 시간 가는줄 모르고 들을 정도로 재미있게 들었다.

나 뿐만 아니라 컨퍼런스 룸에 모여앉아 이야기를 경청했던 모든 사람들도, 코드리뷰 문화에 겁먹지 말고 강연자 분들이 해주신 이야기처럼 적극적으로 다가갈 수 있는 계기가 되었으면 좋겠다. 이슈에서 나누는 이야기들이 너무 어려워 보인다면 관심있게 지켜보는 프로젝트의 문서 한글화 등에 참여해보는 걸로 첫걸음을 떼어도 좋을 것 같고. 어찌되었든 오프라인과 비교 할 수 없을 정도로 가능성이 널려있고, 잘못된다 해도 잃을 것 없이 필요한건 의지와 호기심, 용기 뿐일테니 말이다.

여섯 분의 강연 후 참여자들의 질문을 받아 돌아가며 답변해주시는 자리를 가졌는데, 이는 [별도의 포스트](https://kitchu0401.github.io/2017/oss-dev-story-qna/)에 간략하게 정리한다.

## 발표영상 링크

지난 8월 21일, 별도로 발표영상 링크를 공유받아 추가한다.
- 오픈소스 개발자의 공부방법(강대명): https://youtu.be/svrRov-SDNI
- 나는 오픈소스로 화가가 됐다(김양원): https://youtu.be/WsaTrJJ2G_0
- 나의 오픈소스 이야기(김준기): https://youtu.be/02dxa9NppSY
- Microsoft 오픈소스의 종류와 활용법(김영욱): https://youtu.be/BXqnPVdNcig
- 우아한 오픈소스 이야기(이준호): https://youtu.be/q0i1jBwkdxc
- 오픈소스와 코드리뷰(서주영): https://youtu.be/pkjnpTH0IwU

참고링크
- [페이스북 OSS 개발자럼포럼](https://www.facebook.com/groups/ossdevforum/)
- [온오프믹스 해당 세미나 정보](http://onoffmix.com/event/102936)

[^1]: 오픈소스 생태계에서 프로젝트 오너로서 함께 어울리고 싶다면 잠재적 컨트리뷰터에 대한 배려와 준비가 반드시 있어야한다고 생각(만-_-)한다. 사용자가 필요로 하는 기능은 물론이고 컨트리뷰터가 소스 분석에 어려움을 겪지 않도록 일관성 있는 코딩 컨벤션을 깔끔하게 작성된 코드도 기본사항이라고 생각하며, 오히려 새로운 기능을 덧붙이기 쉽도록 모듈화가 잘 되었는가 여부는 부차적인 문제라고 생각한다.
[^2]: 물론 셀프PR에 불과하며 아직 코드리뷰를 제대로 해본건 아니다. 부끄..