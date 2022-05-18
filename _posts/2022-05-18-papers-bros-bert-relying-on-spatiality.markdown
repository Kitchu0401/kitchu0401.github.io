---
layout: post
title: "[논문 리딩] BROS: A Pre-trained Language Model Focusing on Text and Layout for Better Key Information Extraction from Documents"
date: 2022-05-18 12:37:00 +0900
published: true
comments: true
categories: [paper]
tags: [paper, deep-learning, nlp, kie, AMLM]
---

[BROS: A Pre-trained Language Model Focusing on Text and Layout for Better Key Information Extraction from Documents](https://arxiv.org/abs/2108.04539)을 읽고 내용을 정리합니다. 내용 중 잘못 이해한 내용이 있다면 정정해주시면 감사하겠습니다.

# Abstract

- `KIE from document requires understanding the contextual and spatial semantics in 2D space.`
- `going back to the basic: effective combination of text and layout.`
- BROS(BERT Relying on Spartiality): encodes relative positions of textx in 2D space and learns from **unlabeled documents with area-masking strategy**.

# Introduction

- 통상적인 text sequence 기반 KIE의 파이프라인: OCR - Serialize - KIE
- 텍스트의 올바른 배열에 대한 은탄환이 존재하지 않음. 특히, 다중 칼럼이나 테이블 들에 대해.
- 2D 텍스트 블록의 1D 텍스트 시퀀스로 변환하는 과정에서 공간 구조 정보를 상실하는 문제 → LayoutLM 계열의 모델이 연구됨
- BROS
    - 2차원 좌표값이 아닌 텍스트 블록 간의 상대적 거리를 spartial feature로 활용 (none-visual feature)
    - area-masked language model: 문서 특정 영역의 텍스트를 감추고 모델이 맞추도록 학습
    - 추가로, 아래 두 가지 목표를 중점으로 둠
        1. Text serialization의 의존도 감축 → SPACE decoder를 활용하여 key text blocks를 추출
        2. 학습데이터 라벨링의 의존도(파급력?) 분석?

# Related Work

- LayoutLM, StructuralLM
- BIO tagger
    1. 텍스트 블록 정렬의 의존도
    2. 토큰 레벨 분류(B, I, O)에서는 토큰 간 관계 정보를 활용 할 수 없음

# BERT Relying on Spartiality (BROS)

- 기본 구조는 LayoutLM을 따르나, 아래 두 가지 부분에서 다름
    1. 텍스트 블럭 간 spartial relations을 묘사하는 encoding metric을 활용
        - 기존 LayoutLM처럼 절대좌표가 아닌 텍스트 블럭 간 상대거리를 활용
        - 상대 거리를 측정 할 때 텍스트 블록의 꼭지점을 모두 벡터화하여 비교함으로써 형태와 크기 등의 유사성을 함께 학습, 유사한 key-value 집합을 보다 효율적으로 식별하도록 함
        1. 사인파 함수를 통한 보다 자연스러운 연속된 거리값 인코딩??
        2. Transformer의 어텐션 모듈이 embedding을 공유하도록 함으로써 텍스트 블록 간 spartial relationship을 공유하도록 유도함
    2. 2D 공간을 전제로 한 목적 함수를 활용하여 모델을 학습: Area-masked Language Model (AMLM)
        - 두 가지 목적함수로 모델을 학습: 기존 BERT의 TMLM + 새로운 AMLM (from SpanBERT)
            1. 무작위로 텍스트 블록 선택
            2. BBox를 확장 후 확장된 영역에 포함되는 토큰 확인
                - 영역 확장 여부는 하이퍼 파라미터 λ와 함께 지수 분포에서 샘플링한 값으로 결정
            3. 영역에 포함된 모든 토큰 마스킹 처리
        - AMLM이 공간적으로 근접한 토큰들을 함께 마스킹 처리하기 때문에, 마스킹 된 토큰을 추정하기 위해선 추정 대상 토큰과 이격되어있는 토큰들을 참조하게 됨
    - AMLM으로 15%의 토큰을 마스킹하고, 남아있는 토큰 중 15%를 다시 TMLM으로 마스킹 처리
    - 마스킹 처리는 BERT와 마찬가지로 80%를 `[MASK]` 토큰으로, 10%는 임의의 토큰으로 대치함
- KIE 태스크로 EE(Entity Extraction), EL(Entity Linking)를 활용

# Experiments

1. 토큰 순서 정보 의존성 확인
    - 토큰 순서 정보를 **포함 했을 때**
        - 이미지(LayoutLMv2), 셀(StructuralLM) 등의 추가적인 feature 없이 Text + Layout으로만 학습된 BROS가 타 모델들과 동등하거나 혹은 더 나은 성능을 보여줌
        - 특히, 파라미터 숫자가 3배(110M vs 343M) 많은 LayoutLM Large모델보다 좋은 성능을 보여줌으로써 효과적으로 text / layout feature들을 인코딩했다는 것을 입증함
    - 토큰 순서 정보를 **포함하지 않았을 때**
        - BROS가 LayoutLM, LayoutLMv2 두 모델을 앞서는 성능을 보였으며, 두 모델이 토큰 순서 정보가 포함되었을 때와 비교했을 때 큰 폭으로 성능이 하락한 반면 BROS는 그 하락의 폭이 미미했음
        - 순서 정보가 어떻게 성능에 영향을 주는지 확인해보기 위해 `FUNSD` 데이터셋의 텍스트 블록을 유의미한 순서로 정렬해서 테스트를 수행함
            - LayoutLM, LayoutLMv2 모두 50% 아래의 불안정한 성능을 기록함
            - 흥미롭게도 reasonable한 정렬 순서가 적용된 데이터셋 순서대로 두 모델의 성능 하락폭이 심해졌음 (original - yx 정렬 - xy 정렬 - 무작위 정렬)
            - BROS의 경우 데이터셋에 적용된 정렬 순서와 큰 상관 없이 어느 정도의 성능 수준을 방어해냄
2. 소규모 데이터로 학습 테스트
    모든 경우에서 BROS 모델이 타 모델들에 비해 학습데이터 대비 성능 효율이 좋았음

# Ablation Study

- LayoutLM 모델에 BROS에 적용된 relative positional encoding 및 AMLM을 적용했을 때의 성능 추이 관찰
    - 두 가지 접근을 각각 적용했을 때 모두 유의미한 성능 향상 관찰
- 다음의 인코딩 방식을 적용하여 테스트: 절대 좌표 / 상대 좌표 / BROS의 상대 좌표
    - BROS의 상대 좌표 처리 방식이 가장 좋은 성능을 기록함

# Conclusion

- BROS는 2D 공간에서의 상대적인 좌표 활용과 AMLM 학습을 적용하여 괄목할만한 성능 향상을 이끌어냄
- 실험을 통해 텍스트 블록의 순서 정보와 파인 튜닝시의 학습 데이터의 양에도 강건한 모델임을 입증함

# 요약

- Text + Layout feature만을 사용, 효과적인 인코딩 및 pre-training task 적용을 통해 LayoutLMv2, StructuralLM 등 추가적인 feature를 사용하는 타 모델들의 성능을 뛰어넘음
    - 텍스트 블럭 간 절대좌표 또는 절대거리가 아닌 상대거리를 인코딩
    - TMLM 외 특정 공간의 토큰들을 마스킹하는 AMLM을 적용하여 모델이 인접한 토큰들의 정보를 활용하는 법을 학습하도록 함
- 결과적으로, 기존 text-based KIE 모델에서 오류의 큰 축을 담당하던 text serializer에 대한 의존도를 낮추어 복잡한 구조의 문서에 대한 강건성을 확보하고, feature들의 학습 효율을 증대시킴으로써 적은 양의 학습데이터로도 좋은 성능을 낼 수 있도록 함
