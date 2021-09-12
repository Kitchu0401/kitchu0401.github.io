---
layout: post
title: "AWS CLI를 활용한 s3 정적 웹사이트 배포 자동화"
date: 2021-09-12 17:00:00 +0900
published: true
comments: true
categories: [web-development]
tags: [web, web-application, aws, aws-cli, s3, static-web-site, deploy, automation]
---

다른 취미 등 필요한 일이 생길 때 서버리스 아키텍쳐로 정적 웹 어플리케이션을 코딩해 사용하는 편이다. 컴투스의 모바일 게임 원더택틱스 - 이젠 거진 망한듯; - 를 즐길 때 만들었던 [WonderEquips](https://github.com/Kitchu0401/WonderEquips)라든지, 최근 말리폭스를 한창 즐길 때 유저 토너먼트나 마스터 가이드를 기록하기 위해 만들었던 [사이트](http://malifaux.lazecrew.com/#/guide) 같은게 있었다. 직업 어드밴티지인지 취미에 일정 이상 여력을 붓기 시작하면 자연스럽게 뭔가를 만들게 되는 듯 하다.

첫 시작은 AWS 프리티어의 EC2 컨테이너 상에서 웹 서비스를 띄웠으나, 잊을만 하면 한 번씩 손댈 일이 생기고 그럴 때마다 처음 손대는 일인 양 헤매게 되는 자잘한 관리 공수는 둘째치고 일 년 열두달 취미 코딩을 하는 것도 아닌데 달에 12-20불씩 나가는게 영 마뜩치 않아 최근에는 html 등의 리소스 만으로 정적 웹페이지를 빌드해서 s3 서비스에 정적 자원 호스팅을 통해 배포하고 있다. 산출물들이 비주류 취미와 관련된 페이지들이라 트래픽이 많이 나올 일이야 없었겠지만 s3의 정적 웹 사이트 호스팅은 별다른 이슈가 없는 한 한 달에 1불 미만으로 유지가 가능하다.

다만 소스코드를 빌드 할 때마다 기존의 리소스를 제거하고 일일이 파일 업로드를 하는게 번거로운데, 이번 포스트에서는 AWS CLI를 활용해 s3 버켓에 파일들을 밀어넣는 스크립트를 작성해본다.

## AWS CLI 설치 및 설정

2021년 9월 기준 [공식 도큐먼트](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-cliv2.html)를 참고하여 설치한다. 본인의 경우 윈도우 운영체제를 사용하고 있기 때문에 `msi` 파일을 다운로드해 실행하면 간단하게 설치 할 수 있다.

설치 후 cmd 창에서 아래와 같이 `aws` 명령어 참조가 된다면 정상적으로 설치가 된 것이다.

``` sh
> aws --version
# aws-cli/2.2.37 Python/3.8.8 Windows/10 exe/AMD64 prompt/off
```

설치가 되었음을 확인했으면 당장 사용 할 수는 없고 인증 관련 설정을 진행해야한다. 내가 누구인지 알려주지 않고 API를 할 수는 없으니까. AWS 관리 콘솔의 우측 상단 내 메뉴에서 **내 보안 자격 증명** 항목을 선택, **액세스 키(액세스 키 ID 및 비밀 액세스 키)** 탭에서 **새 액세스 키 만들기**를 선택하여 액세스 키를 생성하고, 무작위 문자열로 생성되는 액세스 키 ID와 보안 액세스 키를 잘 기억한다. 특히 보안 액세스 키는 최초 키 생성시에만 다운로드 또는 직접적으로 확인이 가능하니 키 파일을 잘 보관해두든가 기록해두든가 하자.

액세스 키를 발급 받았다면 `aws configure`를 실행하여 설정을 진행하자.

``` sh
> aws configure
# AWS Access Key ID [None]: {YOUR_ACCESS_KEY_ID}
# AWS Secret Access Key [None]: {YOUR_SECRET_ACCESS_KEY}
# Default region name [None]: ex: ap-northeast-2
# Default output format [None]: ex: json
```

Access Key ID 및 Secret Access Key는 발급받은 액세스 키를 입력하면 되고, region name은 평소 사용하는 AWS 리전 별칭을 입력하면 되며 output format은 AWS CLI 호출시의 응답포맷을 입력하면 된다. [여기](https://docs.aws.amazon.com/cli/latest/reference/#options)를 참고하면 `json`, `text`, `table` 중 하나를 지원하는 듯 하다.

설정이 별 문제 없이 완료되었다면 적당히 `aws s3 ls` 등의 쿼리성 커맨드를 날려 확인해보자.

``` sh
> aws s3 ls
# 2021-08-10 01:46:07 somewhere.you-have.registered
# 2020-06-10 10:11:01 somewhere.you-have.deployed
```

## 정적 웹 리소스 재배포를 위한 프로세스

이제 실제로 빌드된 정적 웹 리소스를 AWS CLI를 통해 s3에 업로드하는 프로세스를 생각해보자.

1. 프로젝트를 빌드한다.
2. (이전에 업로드된 리소스가 존재한다면) 업로드 되어있는 리소스를 모두 제거한다.
3. 새롭게 빌드된 리소스를 업로드한다.

본인은 보통 `vue.js`를 통해 토이 프로젝트를 작성하는 편이라 프로젝트 생성 및 빌드를 위해 [Vue CLI](https://cli.vuejs.org/)를 활용한다. 1번 과정은 `Vue CLI`로 수행하고, 2번 및 3번 과정부터 AWS CLI를 활용해 수행한다. 작업을 수행하는 스크립트는 쉘 스크립트든 자바스크립트든 원하는 쪽을 활용해도 되지만, 개인적으로는 파이썬이 최근 좀 더 친숙하기에 이 포스트에서는 파이썬으로 배포 스크립트를 작성해본다.

2번과 3번 모두 주의해야하는 부분이 있는데, 혹시 다른 방법으로 해결 가능하다면 코멘트 부탁드립니다.

### 업로드 되어있는 리소스 제거

먼저 버킷에 이미 업로드된 리소스 제거를 위해서는 [여기](https://docs.aws.amazon.com/cli/latest/reference/s3/#available-commands)에서 확인 할 수 있는 `s3` `rm` 커맨드를 사용한다. `aws s3 rm {S3_URI}`로 업로드된 리소스를 제거 할 수 있는데, `{S3_URI}`는 기본적으로 `s://{BUCKET_NAME}/{KEY}`의 형식을 띄며 `{KEY}`는 s3 객체의 경로를 넣어주면 된다. 디렉토리의 내용물을 재귀적으로 제거하고 싶다면 `--recursive` 플래그를 추가로 입력해주면 된다.

예를 들어, 업로드된 리소스의 구조가 아래와 같다면

```
root /
    css /
        app.chunk-id.css
        chunk-vendors.chunk-id.css
    js /
        app.chunk-id.js
        chunk-vendors.chunk-id.js.map
    favicon.ico
    index.html
```

아래와 같이 AWS CLI를 호출하면 객체가 삭제되는 것을 확인 할 수 있다.

``` sh
> aws s3 rm s3://my.bucket.url/css --recursive
# delete: s3://my.bucket.url/css/app.chunk-id.css
# delete: s3://my.bucket.url/css/chunk-vendors.chunk-id.css

> aws s3 rm s3://my.bucket.url/js --recursive
# delete: s3://my.bucket.url/js/app.chunk-id.js
# delete: s3://my.bucket.url/js/chunk-vendors.chunk-id.js.map

> aws s3 rm s3://my.bucket.url/favicon.ico
# delete: s3://my.bucket.url/favicon.ico

> aws s3 rm s3://my.bucket.url/index.html
# delete: s3://my.bucket.url/index.html
```

파이썬 코드로는 아래와 같이 작성했다. `aws s3 ls` 커맨드로 업로드된 리소스를 확인하는게 맞지만, 어차피 고정된 구조로 업로드 될 것이기 때문에 빌드된 리소스 목록 그대로 제거하도록 작성했다.

``` py
import glob
import os
import subprocess

S3_BUCKET: str = ...
S3_URI: str = ...
SOURCE_ROOT: str = ...

# 빌드된 리소스 목록을 확인하여 제거한다.
# 디렉토리의 경우 --recursive 플래그를 더해주어야한다.
for file_name in os.listdir(SOURCE_ROOT):
    command: str = f'aws s3 rm {S3_URI}/{file_name}'
    command += ' --recursive' if os.path.isdir(f'{SOURCE_ROOT}/{file_name}') else ''

    # subprocess 모듈을 활용, AWS CLI를 실행한다.
    subprocess.run(command.split())

# delete: s3://my.bucket.url/css/app.chunk-id.css
# delete: s3://my.bucket.url/css/chunk-vendors.chunk-id.css
# delete: s3://my.bucket.url/js/app.chunk-id.js
# delete: s3://my.bucket.url/js/app.chunk-id.js.map
# delete: s3://my.bucket.url/favicon.ico
# delete: s3://my.bucket.url/index.html
```

### 새로운 리소스 업로드

다음으로 빌드된 리소스의 업로드를 위해서는 [여기](https://docs.aws.amazon.com/cli/latest/reference/s3api/#available-commands)에서 확인 할 수 있는 `s3api` `put-object` 커맨드를 사용한다. 기본적으로 s3 버킷에서 객체를 관리하기 위한 `--key` 파라미터로 리눅스 스타일의 경로를 전달해주면 객체가 업로드 될 때 자동으로 디렉토리 형태로 변환해 저장하고, 업로드 된 객체마다 객체 태그를 반환해준다.

``` sh
> aws s3api put-object --bucket my.bucket.url --key favicon.ico --body {SOURCE_ROOT}/favicon.ico
# {
#     "ETag": "\"{TAG_STRING}\""
# }
```

파이썬 코드로는 아래와 같이 작성했다. 윈도우에서 실행하므로 경로 문자열을 편리하게 리눅스 스타일로 변경하기 위해 `pathlib` 모듈을 사용했으며, 별도로 `--content-type` 파라미터를 넘겨주지 않으면 `binary/octet-stream`으로 지정되어 다운로드를 시도하니 확장자를 확인하여 적절하게 추가해주도록 하자.

``` py
import glob
import os
import pathlib
import subprocess

S3_BUCKET: str = ...
S3_URI: str = ...
SOURCE_ROOT: str = ...

# 빌드된 리소스를 모두 개별업로드한다.
for file_path in glob.glob(f'{SOURCE_ROOT}/**/*', recursive=True):
    # glob 모듈이 디렉토리를 포함하여 반환하므로 디렉토리는 제외한다.
    if os.path.isdir(file_path):
        continue

    command: list = [
        'aws', 's3api', 'put-object',
        '--bucket', S3_BUCKET,
        '--body', file_path
    ]
    
    # 윈도우 스타일 > 리눅스 스타일 경로 변환의 편의를 위해 pathlib 모듈을 사용한다.
    path_instance = pathlib.Path(file_path)

    # 리소스 루트 경로는 날려준다.
    # 예: ./{SOURCE_ROOT}/js/app.chunk-id.js > js/app.chunk-id.js
    command += ['--key', '/'.join(path_instance.as_posix().split('/')[1:])]

    # --content-type이 명시되지 않으면 binary/octet-stream으로 지정되므로
    # 파일의 확장자를 확인하여 적절한 --content-type 파라미터를 추가해준다.
    CONTENT_TYPES = {
        '.css': 'text/css',
        '.html': 'text/html',
        '.js': 'application/javascript',
        '.json': 'application/json'
    }

    if path_instance.suffix in CONTENT_TYPES:
        command += ['--content-type', CONTENT_TYPES[path_instance.suffix]]

    subprocess.run(command)

# {
#     "ETag": "\"{TAG_STRING}\""
# }
#
# ...
#
# {
#     "ETag": "\"{TAG_STRING}\""
# }
```
