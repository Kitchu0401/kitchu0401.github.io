---
layout: post
title: "[논문 리딩] Donut: Document Understanding Transformer without OCR"
date: 2022-05-18 12:46:00 +0900
published: true
comments: true
categories: [paper]
tags: [paper, deep-learning, nlp, transformer, kie, qa, vqa]
---

[Donut: Document Understanding Transformer without OCR](https://arxiv.org/abs/2111.15664)을 읽고 내용을 정리합니다. 내용 중 잘못 이해한 내용이 있다면 정정해주시면 감사하겠습니다.

# Abstract

- OCR 모듈에 의존적인 현행 VDU 시스템 → 연산 부하 증가 및 OCR 모듈로부터의 에러 전파
- OCR 모듈을 기반으로 하지 않는 독립적인 end-to-end VDU 시스템을 제언
    - 새로운 학습 태스크
    - 학습시 요구되는 대단위 실제 문서 이미지 생성을 위한 synthetic image generator

# Introduction

- OCR 모듈 기반 VDU 시스템의 문제점
    1. OCR 모델은 학습에도 추론에도 학습 데이터, GPU 자원 등 많은 리소스가 요구되는 아이템임
    2. OCR 모델의 에러는 후속 프로세스로 전파되며, 특히 글자 숫자가 많은 언어에서 심화됨
        이를 해결하기 위하여 correction 모듈을 배치한다 해도 추가적인 비용이 소요됨
- 이를 해결하기 위하여 OCR 및 기타 다른 모듈들이 배재된 독립적인 VDU 모델인 Donut을 제안함
    - 인공 학습 데이터 생성을 위한 문서 이미지 제네레이터인 SynthDoG도 함께 제안

# Method

- OCR 독립적인 심플한 트랜스포머 기반 인코디-디코더 모델 Donut
    - Visual Encoder
        - CNN-based models (ex: ResNet) or Transformer-based models
        - 문서 이미지를 인풋으로 하여 임베딩 집합을 아웃풋으로 함
        - 본 논문에선 encoder 네트워크로 Swin Transformer를 차용
    - Textual Decoder
        - Encoder의 임베딩 집합을 인풋으로 하여 토큰 시퀀스를 나타내는 one-hot vector를 아웃풋으로 함
        - 본 논문에선 decoder 네트워크로 multilingual BART(first 4 layers)를 차용
    - Model Input
    - Output Conversion
- Synthetic Document Generator: SynthDoG
- Task
    - SynthDoG로 생성해낸 인공 이미지 데이터 1.2M건으로 학습
    - 이미지 내 텍스트 시퀀스를 top-left - bottom right로 추출해내는 태스크 설계
- 학습 단계에서 텍스트 시퀀스를 추출해내도록 학습 한 다음, 어플리케이션 구축 단계에서 원하는 형태의 문자열을 반환하도록 파인 튜닝을 수행
    - 논문에서는 1:1 매핑이 가능한 JSON 형태의 정답을 내놓도록 학습함
    - ex 문서 타입 분류: `[START_class][memo][END_class]` → `{"class": "memo"}`

# Expertiments and Analysis

- Document Classification
    - RVL-CDIP 데이터셋: 400K grayscale images, 16 classes, 25K images per class
    - 정의된 N개의 정답 중 하나를 binary confidence로 반환하는 타 모델과 달리, 문서 클래스 정보를 JSON 형태로 표현 가능한 문자열을 생성해내도록 디코더를 학습
    - 절반 가량의 추론 속도에 LayoutLM base모델과 유사한 성능을 확보
- Document Parsing
    ![3](2022-05-16-aicr2.0-papers-3.png)
    - Public 벤치마크 데이터셋(인도네시아 영수증 데이터셋), 자사 데이터셋(일본 명함, 한국 영수증 데이터셋)
    - 문서의 형태를 분석하여 JSON 구조의 문자열로 반환하도록 디코더를 학습
    - normalized Tree Edit Distance로 evaluation
        - 세 가지 데이터셋 모두 BERT-based 모델과 SPADE 모델과 비교하였을 때 확연하게 단축된 추론 속도 및 고수준의 nTED 점수를 확보
- Document VQA
    - DocVQA 태스크: 50K questions on 12K document images
    - BERT base / LayoutLM base / LayoutLMv2 base 모델들보다 낮은 수준의 Averate Normalized Levenshtein Similarity(ANLS)를 기록했지만, 실제 이미지가 포함되지 않은 소규모 인공 데이터를 활용한 것 치고는 준수한 추론 시간과 ANLS를 확보함
    - 10K 정도의 실제 이미지를 학습데이터에 추가하여 인식률이 향상되는 것을 확인 (6p↑)

# Related work

- OCR
- VDU

# Concluding Remarks

- 인풋 이미지에서 원하는 구조의 아웃풋을 바로 추출해내는 end-to-end 모델인 Donut
- OCR 및 large-scale real-world image dataset으로부터 분리됨
- SynthDoG
- "문서를 읽는 방법"에서 시작하여 순차적으로 "문서를 이해하는 방법"을 모델에 학습시키는 접근

# 요약

- 기존 VDU 시스템과 달리 선행되는 OCR 모듈 없이, transformer의 encoder와 decoder만을 두 단계에 걸쳐 학습하여 준수한 성능의 가벼운 VDU 모델을 도출해냈음
- 두 가지 방식을 순차적으로 학습시켜 유의미한 결과를 얻어낸 접근 방식은 흥미로운 반면, 모델이 보여주는 성능은 일부 태스크에 한정적이거나 실제 상업적인 목적으로 적용할만한 수준의 인식률 확보는 아쉬운 수준으로 보임
- 다만 transformer 구조의 단순한 아키텍쳐를 통해 논문에서 의도한 (1) OCR에 독립적인, (2) 실제 이미지 학습 데이터 요구사항 극복이라는 목표는 충분히 달성 된 것으로 보임
