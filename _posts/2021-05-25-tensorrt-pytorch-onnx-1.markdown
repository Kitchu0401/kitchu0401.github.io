---
layout: post
title: "TensorRT #1"
date: 2021-05-25 16:50:00 +0900
published: true
comments: true
categories: [deep-learning]
tags: [deep-learning, nvidia, tensorrt, trial, trouble-shooting]
---

# 도입부

이 시리즈에선 서비스 중인 모델에 TensorRT를 적용한 내용을 기록한다.

일단 NVIDIA TensorRT는 [홈페이지](https://developer.nvidia.com/tensorrt) 대문에서 아래와 같이 설명하고있다.

> NVIDIA® TensorRT™ is an SDK for high-performance deep learning inference. It includes a deep learning inference optimizer and runtime that delivers low latency and high throughput for deep learning inference applications.

> TensorRT-based applications perform up to 40X faster than CPU-only platforms during inference. With TensorRT, you can optimize neural network models trained in all major frameworks, calibrate for lower precision with high accuracy, and deploy to hyperscale data centers, embedded, or automotive product platforms.

어쨌든 이미 빌드된 모델을 더욱 빠르게 돌릴 수 있게 해준단다. ㄹㅇ? 라는 말이 튀어나오지만 실제로 주변에 효과를 보았다는 지인이 있어 첫걸음을 조금 더 희망차게 뗄 수 있겠다. 팀에서 시도하는 여러 접근법 중 내가 담당한 부분은 `torch` 기반 모델을 `onnx`를 거쳐 TensorRT로 포팅 해보는 것. 실제 모델 개발 및 딥한 내용과는 거리가 있고 절차와 트러블슈팅에 집중하여 기록할 예정이다.

# 설치

TensorRT 관련된 포스팅이 다양하지 않아 서칭하다보니 결국 몇몇 게시글로 귀결된다는 느낌을 받았는데, 개인적으로는 주로 [이곳](https://si-analytics.tistory.com/32)을 참고했다. 설치부터 적용까지 아주 깔끔하게 정리해두셔서 뭐 덧붙일 내용이 별로 없다.

## 버전 확인

적용 대상 서비스의 환경이 아래와 같으므로

- CUDA: `10.0`
- CUDNN: `7.4.2`
- NVIDIA Graphic Driver: `430.14`

[이곳](https://docs.nvidia.com/deeplearning/tensorrt/support-matrix/index.html)을 참조하여 적용할 TensorRT의 버전을 결정한다. 링크한 도큐먼트는 최신 버전이고, 내 경우처럼 올드한 버전에 적용하려면 [Archive 페이지](https://docs.nvidia.com/deeplearning/tensorrt/archives/index.html)에서 일일히 TensorRT Support Matrix 페이지를 확인하여 버전을 체크해야한다.

버전을 체크했다면 설치는 알아서 - 난 설치파일을 직접 다운로드 하는 방식을 선택했다. 아래의 코드 중 버전 및 실제 파일명을 참조하는 구문은 당연히 사람마다 다를 수 있겠다.

## 다운로드 및 설치

``` sh
# tar.gz 다운로드
wget https://developer.download.nvidia.com/compute/machine-learning/tensorrt/secure/7.0/7.0.0.11/tars/TensorRT-7.0.0.11.Ubuntu-16.04.x86_64-gnu.cuda-10.0.cudnn7.6.tar.gz

# 압축해제
chmod +x TensorRT-7.0.0.11.Ubuntu-16.04.x86_64-gnu.cuda-10.0.cudnn7.6.tar.gz
tar xzvf TensorRT-7.0.0.11.Ubuntu-16.04.x86_64-gnu.cuda-10.0.cudnn7.6.tar.gz
```

이후 `LD_LIBRARY_PATH` 환경변수에 압축이 풀린 경로를 포함하여 - 이를테면 - 아래와 같이 추가한다. 아마 다른 딥러닝 패키지를 사용중이라면 으레 설정해두었을 환경변수일거라 생각한다. 아니면 각자 알아서들 하시고.

```
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/home/someuser/TensorRT-7.0.0.11/lib"
```

### Troubleshooting

`ImportError: libnvinfer.so.7: cannot open shared object file: No such file or directory`

- ref: [ImportError: libnvinfer.so.4: cannot open shared object file: No such file or directory](https://forums.developer.nvidia.com/t/importerror-libnvinfer-so-4-cannot-open-shared-object-file-no-such-file-or-directory/62243)
- `LD_LIBRARY_PATH` 환경변수에 설치된 경로의 `TensorRT-your.version/lib`로 정확하게 경로를 추가해준다.

## 확인

`import tensorrt` ㄱㄱ
