---
layout: post
title: "미완성 한글의 ORDER BY에 대하여"
date: 2020-07-10 18:21:00 +0900
published: true
comments: true
categories: [database]
tags: [database, mariadb, mysql, oracle, orderby, korean, abnormal-korean]
---

메이저한 언어의 경우 데이터베이스의 COLLATION 이라는 개념으로 정렬에 대응이 가능하지만 최근 미완성 한글이 포함된 케이스에서 정렬이 제대로 되지 않는 이슈가 있어 기록차 적어둔다. 한자와 미완성 한글이 혼재하는 케이스가 어디 흔하겠냐만 배 째고 나가기엔 찝찝하니까..

대충 테스트를 위한 테이블을 생성해보자. mariadb 기준으로:

``` sql
DROP TABLE IF EXISTS T_TEST_CHARSET;
CREATE TABLE T_TEST_CHARSET (
	`TEXT` varchar(50) NOT NULL
) DEFAULT CHARSET=utf16;

INSERT INTO T_TEST_CHARSET VALUES ('a');
INSERT INTO T_TEST_CHARSET VALUES ('ㄻ');
INSERT INTO T_TEST_CHARSET VALUES ('发');
INSERT INTO T_TEST_CHARSET VALUES ('가');
INSERT INTO T_TEST_CHARSET VALUES ('힣');
```

정렬을 적용한 간단한 쿼리를 날리면 생각했던대로 결과가 나타나지 않는다.

``` sql
SELECT TEXT, ORD(TEXT), ASCII(TEXT), HEX(TEXT)
  FROM T_TEST_CHARSET
 ORDER BY TEXT

-- a	97      0	0061
-- ㄻ	12603	49	313B
-- 发	21457	83	53D1
-- 가	44032	172	AC00
-- 힣	55203	215	D7A3
```

COLLATION이나 `BINARY()`, `CONVERT()` 등을 먼저 시도해봤지만 효과가 없음. 미완성 한글의 경우 있어야할 자리가 아닌 한자 앞으로 훌쩍 건너가버린다. 머하냐 진짜.

구글링을 조금 더 해보니 [여기](https://blog.naver.com/PostView.nhn?blogId=cuteclare&logNo=80181518356&proxyReferer=https:%2F%2Fwww.google.com%2F)나 [여기](https://blog.naver.com/troopa102/120168125986)처럼 언어별로 `CASE WHEN` 구문을 활용해 해결을 본 케이스가 있다. 링크의 코드들이 미완성 한글 케이스를 해결해주는 것은 아니지만, `CASE WHEN`을 활용하여 해결을 봤다. 쿼리문이 좀 지저분해지기는 해도 mariadb 및 oracle 모두 코드 베이스로 해결이 가능하다.

``` sql
SELECT TEXT
  FROM T_TEST_CHARSET
 ORDER BY CASE WHEN (TEXT BETWEEN '0' AND '9') THEN 0
               WHEN (TEXT BETWEEN 'a' AND 'Z') THEN 1
               WHEN (TEXT BETWEEN 'ㄱ' AND 'ㅎ') OR (TEXT BETWEEN 'ㅏ' AND 'ㅢ') OR (TEXT BETWEEN '가' AND '힣') THEN 3
               ELSE 2 END ASC, TEXT ASC

-- a
-- 发
-- ㄻ
-- 가
-- 힣
```

charset으로 해결 볼 수 있는지 모르겠다. `utf8_general_ci`, `utf8_bin`, `utf8mb4`, `utf16` 이것저것 모두 시도해봤는데 안되더라. 지나가시는 분 중에 지저분한 쿼리문 말고 해결방법이 있으면 좀 알려주고 가세요.
