---
layout: post
title: "[논문 리딩] Survey of Post-OCR Processing Approaches"
date: 2022-06-14 16:43:00 +0900
published: true
comments: true
categories: [paper]
tags: [paper, survey, deep-learning, nlp, ocr, post-processing, error-detection, error-correction]
---

[Survey of Post-OCR Processing Approaches](https://dl.acm.org/doi/abs/10.1145/3453476)을 읽고 내용을 정리합니다. 내용 중 잘못 이해한 내용이 있다면 정정해주시면 감사하겠습니다.

# Abstract

- OCR의 문제점: 다양한 환경에서 하락하는 퍼포먼스
- OCR 결과를 개선하기 위한 후처리의 중요성과 그 방안, SOTA 접근법 등을 다룸

# Introduction

- 물리적인 문서의 디지털화 기법 선택에는 활자 / 손글씨 여부, 언어 등의 다양한 요소가 영향을 미치며, 통상적으로 수기로 직접 옮기거나, OCR 소프트웨어를 활용함
- 비용과 보안 측면에서 수기 입력보다 OCR이 더 효율적
- OCR의 퍼포먼스는 문서의 재질, 복잡한 구조, 오래된 글씨체 등에 의해 크게 영향을 받으며, 디지털화된 텍스트를 입력으로 사용하는 하위 어플리케이션(IR: Information Retrieval, NLP: Natural Language Processing)의 퍼포먼스에도 영향을 미침
    - 글자 단위 에러보다 단어 단위 에러가 퍼포먼스 하락에 더 큰 영향을 미치는 경향이 있음
- Contribution:
    - OCR 후처리 문제를 정의하고 통상적인 파이프라인에 대해 다룸
    - OCR 후처리 접근방안을 다루고, 분류 후 장단점을 다룸
    - Evaluation metric과 접근 가능한 데이터셋을 다룸
    - OCR 후처리 분야의 최근 트렌드와 연구방향을 다루고

# Definition of Post-OCR processing problem

- OCR 결과 토큰 `S=s1,s2,s3,..,sm`이 있을 때, 원본 문서에 기재된 실제 단어 집합 `W=w1,w2,w3,..,wn`을 찾는 것
    - Segmentation 오류 등의 이유로 `len(S) != len(W)`
- 대부분의 접근은 순차적으로 OCR 결과 토큰 `s`가 `w`를 선택하도록 함
    - `s == w`: 올바른 OCR 결과
    - `s != w`: 잘못된 OCR 결과
        - `s`가 실존하는 단어가 아니라면 `non-word` 오류
        - `s`가 실존하는 단어라면 `real-word` 오류: 문맥 오류를 의미
- `P(w|s)` 및 `P(w1,w2,w3,..,2n)`가 최대가 되도록 학습 → 문맥에 걸맞는 단어와 매칭하는 방식으로 가급적 많은 단어들이 원래 단어와 일치되는 선택을 할 수 있도록
- 에러 감지와 에러 교정 파이프라인:
  1. OCR
  2. 에러 감지
  3. 에러 후보군 추출
  4. 에러 후보군 정렬
  5. 후처리된 OCR 텍스트

# Post-OCR Processing Techniques

- 본문에 기재된 논문들의 수집 및 분류 절차
  1. `OCR post`, `OCR correct`, `OCR detect` 키워드로 2020년 12월까지의 논문을 DBLP 및 Google Scholar에서 수집
  2. 후처리 관련된 내용이 기술되어있는지 확인 후 수동 / 반자동 / 자동 기법으로 분류
  3. 연관 논문들도 목록에 포함

![1](2022-06-14-papers-survey-of-post-ocr-processing-approaches-1.PNG)

## 1. Manual approaches

ReCAPTCHA

## 2. (Semi-)automatic approaches

- 2-1. Isolated-word approaches: OCR 결과 토큰만 피쳐로 활용
  - 사전 베이스 유사도(lavenstein?), 등장 빈도, OCR confidence 등
  - non-word 에러 위주로 교정 및 수정을 시도함
- 2-2. Context-dependent approaches: OCR 결과 토큰 및 주변 문맥도 함께 활용
  - word ngram 언어 모델, 품사 태깅 등
  - non-word 에러와 real-word 에러를 모두 대응 가능

### 2-1-1. Merging OCR outputs

1. 복수의 OCR 결과물을 병합
    - 동일한 엔진으로 여러 번 스캔 및 OCR 돌린 결과물을 병합
    - 여러 버전의 binarization을 적용한 결과물을 병합
    - 복수의 엔진들의 결과물을 병합

2. 병합된 결과물을 정렬
    - 동일한 책의 다른 스캔본이나, 다른 버전의 스캔본으로 sequence alignment를 생성
    - A* 알고리즘 [Creating an improved version using noisy OCR from multiple editions.](https://ciir-publications.cs.umass.edu/getpdf.php?id=1104)
      - 최단 경로 탐색을 위한 그래프 알고리즘으로써, 휴리스틱 추정값을 통해 최적화 가능 [[참고](http://www.gisdeveloper.co.kr/?p=3897)]
    - Weighted Finite-State Transducer (WFST) [[참고](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.447.2029&rep=rep1&type=pdf)]
    - Line-to-Page alignment: Weighted Finite-State Transducers (WFST)
      - [Combination of multiple aligned recognition outputs using WFST and LSTM](https://www.diva-portal.org/smash/get/diva2:1271586/FULLTEXT01.pdf)
      - > OCR system A의 각 line을 OCR system B의 전체 페이지를 text source로 하는 WFST에 통과시킴으로써 recognition 및 segmentation 에러를 교정하고자 하는 접근

3. 최종 결과물 선정
    - Voting 알고리즘
    - Long Short-Term Memory (LSTM) [Combination of multiple aligned recognition outputs using WFST and LSTM](https://ieeexplore.ieee.org/document/7333720)
    - Conditional Random Fields (CRF) [Combining multiple thresholding binarization values
to improve OCR output](https://www.semanticscholar.org/paper/Combining-multiple-thresholding-binarization-values-Lund-Kennard/caeca18c96c8be6bcac9be775bab584b5b1b78d7)

- 이러한 접근방식은 단일 OCR 엔진을 활용 했을 때보다 좋은 결과를 보여주었으나, 문맥 정보를 고려하지 못해 real-word 에러를 수정하지 못하고 OCR 엔진을 여러 차례 태워야하기 때문에 시간 및 비용적으로 실용적이지 못함.

### 2-1-2. Lexical approaches

사전 및 ngram 기반의 후처리 접근: 사전에 등재된 단어들과 캐릭터 또는 ngram 기반의 edit distance를 비교하여 최적의 correction candidates를 추출

- Damerau-LV distance: single-character edit operations or ngram distance
- Refinement of the LV distance based on frequent error patterns
- 사전의 볼륨에 영향을 받는 경우가 있었기 때문에, 특정 도메인의 코푸스를 수집하여 보충하거나 하는 연구가 있었음
- [Distributional Similiarity](https://qr.ae/pvHHiA): Word2Vec, Glove

(LV distance: Levenshtein edit distance)

### 2-1-3. Error models

- [OCR error correction using a noisy channel model](https://dl.acm.org/doi/pdf/10.5555/1289189.1289208)
  - Noisy channel 모델 및 구문 패턴 인식을 위한 프레임워크를 통해 OCR 후처리를 접근
- 확률적 finite-state machine을 활용한 에러 모델
  - Weighted Finite-State Transducers (WFST), Hidden Markov Model (HMM) 등의 기법을 활용
  - Weights used in WFST: 
    1. 인풋으로 글자가 주어졌을 때 동일한 문맥에서 등장하는 글자들의 빈도수를 계산
    2. 인풋 - 아웃풋 또는 아웃풋 - 아웃풋 글자 간의 관계를 고려하는 퍼셉트론 태거에 의해 계산
      - [Optical character recognition with neural networks and post-correction with finite state methods](https://www.researchgate.net/publication/343762421)
      - > ?
- Historical documents에서 local / global information을 프로파일링하여 후처리에 활용하는 접근
  - Local profile: GT 및 historical spelling 이력을 기반으로 한 correction candidates suggestion
  - Global profile: OCR 에러 타입과 historical patterns
- Regularities Based Embedding of Language Structures (REBELS)
  - Noisy word(error)와 referenced word(words in dictionary)가 어떤 독립적인 구조(접사: infix)로? 엮여있는지?
  - 독립적인 접사(distinctive infix): 접두사 혹은 둑립적인 두 문맥을 이어주는 접사를 뜻함
    - > 부자연스러운 문맥이 존재 할 때 이러한 부자연스러움이 단어 에러로 인해 발생했다고 가정하면, 정상적인 문맥으로 복원하기 위한 과정에서 character variations for correction을 추출 할 수 있을 것이라 가정하는 듯

### 2-1-4. Topic-based language models

- Correction candidates를 필터링하기 위해 문서의 토픽 정보를 활용함
- 교체될 단어를 선택 할 때 토픽 별 교체될 단어의 등장 빈도도 함께 고려하도록 함

### 2-1-5. Other models

- Feature-based machine learning, string-to-string transformation, neural machine translation (NMT)
- Support vector machine (SVM): Supervised OCR post-correction of historical swedish texts: what role does the OCR system play?
  - 아래 일곱 가지의 피쳐를 활용한 SVM으로 OCR 에러를 탐지함:
    1. 단어 내 non-alphanumeric 글자 수
    2. 단어 내 모음 수
    3. 단어 내 대문자 수
    4. Tri-gram 글자 빈도
    5. 단어 내 숫자 등장 여부
    6. 단어 등장 빈도
    7. 단어 길이
  - 이후 사전에서 LV distance 기반 일정 이하 threshold를 가진 단어들을 correction candidates로 제시함

- *semi-auto 방식이라 모두 skip함*

### 2-2-1. Language models

#### Statistical language models

- 단어 시퀀스의 확률 분포를 측정하는 통계적 언어 모델을 활용하는 접근
- 확률적 캐릭터 에러 모델 + WFST (RAE in ICDAR2019): [ICDAR 2019 competition on post-OCR text correction](https://hal.archives-ouvertes.fr/hal-02304334/document)

#### Neural network-based language models

- Word embedding 기반, 통상적으로 주어진 문맥을 활용하여 다음 단어 이후의 확률 분포를 예측하는 모델로 학습
- 단어 레벨 / 글자 레벨 모두 적용 가능
- 글자 레벨의 양방향 LSTM 언어 모델을 활용한 프랑스어 후처리 연구
  - 에러가 없는 깨끗한 텍스트가 주어졌을 때, 삭제/삽입/수정 등 무작위로 수정을 가하여 학습데이터를 생성하는 방안 포함
  - [Low-resource OCR error detection and correction in french clinical texts](https://hal.archives-ouvertes.fr/hal-01831225/document)
  - [Generating a training corpus for ocr post-correction using encoder-decoder model](https://hal.archives-ouvertes.fr/hal-01831147/document)
  - *The authors also note that their models are not good at detecting errors related to word boundary.*

#### Feature-based machine learning models

아래는 regression model로 correction 후처리를 해결하려던 접근에서 활용한 feature들

- Confusion weight(from training data), word frequency, forward/backward word bigram frequencies (+ OCR confidence, term frequency)
  - (X) 때때로 OCR confidence가 제공되지 않음
- Similarity metrics(OCR 에러의 특성과 관련된), word frequency, 도메인 디펜던시의 단어 존재 여부, word ngram frequency, word skip-gram frequency
  - (X) No confusion weight
- Non alpha-numeric 글자 존재 여부, 사전에 존재하는 단어 존재 여부, 일정 기준치 이상의 token / context frequency, word bigram frequency
- (중략) frequency of an OCRed token in its candidate generation sets, modified version of a peculiar index
- 에러 감지 및 대체 단어 후보군 생성을 위한 OCRSpell, 후보군 랭킹을 위한 SVM
  - Edit distance, confusion weight, unigram frequency, backward/forward word bigram frequencies

#### Sequence-to-sequence (Seq2Seq) models

OCR 후처리 과정을 기계독해(machine translation: MT)로 접근하는 방안

- [Using SMT for OCR error correction of historical texts](https://www.researchgate.net/publication/301552274): large dataset

#### Neural network Seq2Seq models (or NMT-based models)

- Character-level Seq2Seq 모델을 활용하여 segmentation(띄어쓰기?) 에러를 해결함
  - 공백을 `##`으로 치환?
  - [Correction of OCR word segmentation errors in articles from the ACL collection through neural machine translation methods](https://aclanthology.org/L18-1113.pdf)
- Context-based Character Correction (CCC)
  - ICDAR2019 competition 내 word correction 분야 우승한 방안
  - Bidirectional Encoder Representations from Transformers: BERT + convolution layers + fully connected layers → OCR 에러 탐지

# Evaluation Metrics, Datasets, Language Resources, and Toolkits

## TP, FP, TN, FN, Precision, Recall, F-score

에러 감지 태스크의 목표는 OCR 결과 토큰이 올바르게 인식되었는지 여부를 판단하는 것이며, 이는 곳 이진 분류 문제

- True positives (TP): 올바르게 탐지된 에러의 숫자
- False positives (FP): 에러가 아니지만 에러로 탐지된 올바른 단어들
- False negatives (FN): 탐지해내지 못한 에러의 숫자
  
  ![2](2022-06-14-papers-survey-of-post-ocr-processing-approaches-2.PNG)

OCR 후처리 태스크의 케이스에 맞게 변형하면 (T/F: GT와 일치 여부, P/N: 수정 여부):
  - 틀린 글자/단어가 후처리를 통해 올바르게 수정된 케이스: TP
  - 올바른 글자/단어가 후처리를 통해 틀리게 수정된 케이스: FP
  - 올바른 글자/단어가 후처리를 통해 올바르게 유지된 케이스: TN
  - 틀린 글자/단어가 후처리를 통해 틀리게 유지된 케이스: FN

## Bilingual evaluation understudy: BLEU

- 번역된 텍스트가 Ground truth 텍스트와 얼마나 유사한지를 판별하는 척도
- [BLEU: a method for automatic evaluation of machine translation](https://dl.acm.org/doi/pdf/10.3115/1073083.1073135)

![3](2022-06-14-papers-survey-of-post-ocr-processing-approaches-3.PNG)

- Relative Improvement: RelImp
- Average Corrected Distance: avgDist
- Character Error Rate: CER
- N: GT에 포함된 전체 글자 수

# Discussion

- Isolated-word 계열은 포화 상태
- Context-dependant 계열은 딥러닝 테크닉 등에 힘입어 우세
- 관련된 착안점들.. (후략)
  - > 추후 시간 내어 훑어 볼 것

# Conclusion

본문에서 다루고 있는 내용들에 대한 회고

- OCR 후처리에 대한 태스크 정의와 통상적인 프로세스 파이프라인
- OCR 에러의 하위 프로세스에 대한 파급력으로 알아보는 OCR 후처리의 중요도
- 정보의 활용 범위 및 활용되는 테크닉에 따른 OCR 후처리 접근 방안 세분화
- Evaluation metrics 및 데이터셋 정리

# Reference links

- [Combination of multiple aligned recognition outputs using WFST and LSTM](https://ieeexplore.ieee.org/document/7333720)
- [ICDAR 2019 competition on post-OCR text correction](https://hal.archives-ouvertes.fr/hal-02304334/document)
  - The competition team from Centro de Estudios de la RAE
- [Correction of OCR word segmentation errors in articles from the ACL collection through neural machine translation methods](https://aclanthology.org/L18-1113.pdf)
- ..
