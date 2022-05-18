---
layout: post
title: "[논문 리딩] One-shot Key Information Extraction from Document with Deep Partial Graph Matching"
date: 2022-05-18 11:35:00 +0900
published: true
comments: true
categories: [paper]
tags: [paper, deep-learning, nlp, graph, kie, one-shot-learning]
---

[One-shot Key Information Extraction from Document with Deep Partial Graph Matching](https://arxiv.org/abs/2109.13967)을 읽고 내용을 정리합니다. 내용 중 잘못 이해한 내용이 있다면 정정해주시면 감사하겠습니다.

# Abstract

- One-shot KIE using partial graph matching
- DKIE dataset (2.5K, from mobile phones)

# Introduction

- 기존의 KIE 프로세스는 문자 인식 - 문자 분류 - 필드 라벨링
- 필드 라벨링에서의 one-shot learning 기법은 많은 연구 x
- supervised learning 연구는 많이 진행되었으나, 데이터 확보에 많은 시간과 노동력 요구됨
  one-shot learning 연구가 진행되었으나, 기존의 연구는 generalize에 실패함
- covering drifted field: 이상한 위치의 field / outliers: 학습데이터의 패턴과 상이한 field
- position + shape + text embedding -> most high summed similarity
- 1:1 매칭
- DKIE 데이터셋 상에서 SOTA 달성?

※ one-shot KIE with partial graph matching, 1:1 constaints, dealing with documents ontaining drifted / outliers fields

# Related Works

- KWS
- KIE
- One-shot learning of KIE
- Graph Matching

# Our Model

- partial graph matching problem formulated
    ![1](2022-05-16-aicr2.0-papers-1.png)
    - (2)의 두 제약조건으로 다:다 매칭 및 매칭되지 않는 케이스가 배재
    - (1)의 좌측: 노드 간 similarity score를 구하기 위하여 positive case를 모두 더함
        - sums over all possible matching between support and query fields to calculate the vertex similarity score
    - (1)의 우측: 간선 간 similarity score를 구하기 위하여 positive case를 모두 더함
        - sums over all possible matching between support and query edges to calculate the edge similarity score
- Fig 4.에서:
    - (a) 정답이 다중 행에 걸쳐있을 때, 1:1 매칭 제약을 충족하기 위해 label에 숫자 suffix를 붙여 추출 후 후처리로 suffix를 제거함
    - (b) / (c) drifted / outlier 이미지에서 1:1 매칭 제약으로 인해 모델이 여러 텍스트 대신 그 중 최적의 정답 하나를 답으로 내놓음
- Document Graph Construction
    - spartial features: line segments connecting center points of Landmarks (header texts) with a single field (value texts)
    - aspect features: width and height of Fields (value texts)
    - textual features: average word embedding with 300 dimensions
- Graph edges
    - build a visible graph among fields - Prime algorithm - minimum spanning tree
- Combinatorial Solver
    - solving partial graph matching problem
        - DD-ILP solver
        - ZAC-GM solver
    - back propagate through solver
        - Hamming Loss - generates gradients even for wrong prediction

# Experiments

- DKIE One-shot Dataset - one-shot KIE task focusing on drifted / outlier cases
    - `However, each style needs independent support document because different styles have different landmarks and labels for the on-shot learning methods.`
- Implementation Details
    - supervised-learning-based-models:
        - 스타일 당 3천장의 문서(생성된 이미지 포함)를 학습데이터로 사용
        - `We generate 3,000 documents for each style, including all the real samples, and the rest are synthesized.`
    - one-shot-learning-based-models (LF-BP, our model):
        - 하나의 모델로 여러 스타일의 문서들을 처리
        - support로 지정된 문서가 많은 landmarks/fields를 가질 수록 query 문서에 대한 추출율이 높아지는 경향
- Experimental Results
    - LF-BP 모델과의 형평성을 위하여 **one-shot 모델에는 spartial feature만 사용**
    - drifted / outlier cases에서 LF-BP모델을 압도함
    - our model 중에선 DD-ILP solver보다 ZAC-GM solver가 outlier case에서 더 좋은 성능을 보여줌
    - drifted / outlier cases가 섞여있는 데이터셋에서 our model의 성능 하락을 관찰했는데, spartial feature만으로는 둘의 판별이 어려움
    - spartial feature만 사용했는데도 supervised-learning-based-model과 비슷한 수준의 성능을 보임
        - supervised-learning-based-model은 본 실험에서 아래의 어드밴티지를 가지며
            1. one-shot-learning-based-model과 비교해 훨씬 많은 학습데이터가 투입됐고
            2. one-shot-learning-based-model과 비교해 문서 스타일 당 학습을 진행했음
        - 반례로, supervised-learning-based-model은 새로운 문서 스타일에 대한 추출 진행시 별도의 학습데이터 라벨링이나 학습 자체가 불필요하며, support 문서만 올바르게 함께 투입 될 경우 준수한 추출율을 확보 할 수 있음
- Ablation Study
    - Different features: spartial, aspect, text, edge 네 feature들은 상호보완적임
    - Landmarks: spartial, aspect, edge 세 feature들만 사용한 상태에서 landmark를 좀 drop해도 (~-3) 성능이 유지됨. 고로 이 세 feature들 역시 상호보완적임
    - Ranking loss: ranking loss를 제외해도 LF-BP모델보다 성능이 좋았으나, ranking loss를 적당히 더해주면 성능도 개선되며 학습 시간도 단축되는 효과

# Conclusion

- A One-shot framework that combines deep learning + combinatorial solver, can handle drifted / outlier cases.

# 정리

- 실제 KIE 태스크에서 마주치는, 해결이 어려운 케이스(drifted / outlier)에 강건하고 새로운 유형의 문서 마다 신규 학습이 요구되지 않는 one-shot KIE model with partial graph matching을 제안함
    - 특히, partial graph matching에 적용된 1:1 매칭 제약조건으로 인해 drifted / outlier 케이스에서 높은 수준의 인식률을 보여주고
    - 새로운 유형의 문서라 할지라도 query document(추출 대상 문서)들과 함께 올바른 support document(템플릿 문서?)가 제공된다면 추가 학습 없이 준수한 성능 유지 가능
- 모델에서 사용하는 feature:
    - spartial feature: 개별 필드(추출 대상 텍스트 블럭)마다 랜드마크(헤더 텍스트 블럭)와의 중앙점을 연결하는 line segment
        - key - value 텍스트 블럭 간 각도?
    - aspect feature: 개별 필드 텍스트 블럭의 너비 및 높이
        - key에 해당하는 value 텍스트 블럭의 형태적인 유사성 (length of value)
    - textual feature: average word embeddings with 300 dimensions
    - edge feature: ?? 1:1 관계를 유지하는 최소 간선 트리..?
