---
layout: post
title: "TensorRT #2"
date: 2021-06-03 16:50:00 +0900
published: true
comments: true
categories: [deep-learning]
tags: [deep-learning, nvidia, tensorrt, trial, trouble-shooting]
---

TensorRT를 적용하기 위한 다른 방법들은 대충 아래의 방법들이 있다. 또 다른 구현체 / 래퍼들이 존재 할 수 있음.

- https://github.com/nightduck/keras2trt
- https://github.com/NVIDIA-AI-IOT/torch2trt

 

내가 선택한 방법은 구현체를 사용하지 않고 `torch` 모델을 `onnx`를 거쳐 TensorRT로 사용 가능하게 변환하는 방법이다. 이 포스트에서는 TensorRT로 변환 전 `onnx`로의 변환 과정을 먼저 다룰 예정이며, 전반적인 프로세스는 `torch` 공식 도큐먼트의 [`torch.onnx` 항목](https://pytorch.org/docs/1.8.1/onnx.html)과 [파이토치 한국 사용자 모임의 튜토리얼](https://tutorials.pytorch.kr/advanced/super_resolution_with_onnxruntime.html)을 참고했고 특수한 이슈나 트러블 슈팅을 집중적으로 기록할 예정이다.

관련 패키지 버전은 아래와 같다.

- `torch`: `1.5.0`
- `torchvision`: `0.7.0`
- `onnx`: `1.8.1`
- `onnxruntime`: `1.7.0`

## 설치

`torch`, `torchvision` 설치는 생략함. 

`onnx` 설치도 별거 없다.

``` sh
# Anaconda 가상환경 설치
conda install -c conda-forge onnx

# 난 그냥 pip로 설치했다.
python -m pip install onnx
```

`onnxruntime`의 경우도 별거 없을 줄 알았는데, [Installtion 항목](https://www.onnxruntime.ai/docs/how-to/install.html)을 보면 CPU와 GPU 버전이 분리되어있으니 참고하자. 내 경우는 TensorRT로 변환 전 중간 과정이 올바른지 테스트를 위함이라 CPU 버전으로 설치했다.

``` sh
# CPU
python -m pip install onnxruntime

# GPU
python -m pip install onnxruntime-gpu
```

## 변환

모델을 로드하여 변환한다. 보안 이슈상 실제로 변환한 모델의 코드는 당연히 올리지 못하니 모델 로드 부분은 [파이토치 한국 사용자모임의 튜토리얼 코드](https://tutorials.pytorch.kr/advanced/super_resolution_with_onnxruntime.html#pytorch-onnx-onnx)로 대신하자. 변환 전 추론 모드로 전환하라는 아래의 내용에 주의.

> 모델을 변환하기 전에 모델을 추론 모드로 바꾸기 위해서 torch_model.eval() 또는 torch_model.train(False) 를 호출하는 것이 중요합니다. 이는 dropout이나 batchnorm과 같은 연산들이 추론과 학습 모드에서 다르게 작동하기 때문에 필요합니다.

``` py
# model 선언
torch_model = SuperResolutionNet(upscale_factor=3)

# 학습된 가중치 load
torch_model.load_state_dict()

# 모델을 추론 모드로 전환
torch_model.eval()
```

`torch.onnx` 패키지의 `export()` 함수로 `onnx` 바이너리 파일을 생성한다. 

``` py
import onnx
import torch

arguments = (torch.as_tensor(np.ones([1, 3, 960, 640]), dtype=torch.float32).cuda())
output_path_onnx = 'model.onnx'
input_names = ['inputs']
output_names = ['outputs']
dynamic_axes = {'inputs': {2: 'h', 3: 'w'}, 'outputs': {2: 'h', 3: 'w'}}

torch.onnx.export(model=torch_model,          # 실행될 모델
                  args=arguments,             # 모델 입력값 (튜플 또는 여러 입력값들도 가능)
                  f=output_path_onnx,         # 모델 저장 경로 (파일 또는 파일과 유사한 객체 모두 가능)
                  input_names=input_names,    # 모델의 입력값을 가리키는 이름
                  output_names=output_names,  # 모델의 출력값을 가리키는 이름
                  dynamic_axes=dynamic_axes,  # 가변적인 길이를 가진 차원
                  export_params=True,         # 모델 파일 안에 학습된 모델 가중치를 저장할지의 여부
                  opset_version=11            # 모델을 변환할 때 사용할 ONNX 버전
)
```

- 동적 input을 가지는 이미지 처리 모델에 대한 변환 코드다.
- 동적 input을 처리하기 위하여 `dynamic_axes` 파라미터를 정해 넘겨준다.
  - `dynamic_axes`의 키 값에 해당하는 `h`, `w`는 임의로 넣어주었다. 키 값이 중요한듯. (input layer shape index?)
  - `dynamic_axes`가 정의되어 있더라도 `input_names`에 매핑되지 못하면 적용되지 않는다.
  - `dynamic_axes`의 `outputs`는 내 경우는 큰 의미가 없었다.
- `opset_version`의 경우 변환하고자 하는 레이어에 대한 operation set이라고 이해하면 편하다.
  - [여기](https://github.com/pytorch/pytorch/tree/master/torch/onnx)서 opration set 별 매핑된 정보를 확인 할 수 있는 듯 하다.
  - 변환하고자 하는 모델에 따라 `opset_version`을 적절하게 변경해주어야 할 수 있다.
- 변환한 `onnx` 바이너리를 통해 추론시 추가로 가중치를 로드하지 않을거라면 - 사실 가능한지도 모르겠고 시도해볼 생각도 없다 - `export_params=True`로 주어 모델 가중치를 바이너리에 포함시키도록 하자.

## 추론

`onnxruntime` 및 변환한 바이너르 파일로 `InterenceSession`을 초기화하고, 이를 통해 추론을 수행한다. 추론 결과에 대한 후처리 및 원본 모델과의 cross checking 과정은 생략함.

``` py
import onnxruntime
import torch

session = onnxruntime.InferenceSession('model.onnx')

# 사전에 numpy.array로 변환된 이미지를 input으로 사용
x = torch.from_numpy(image).to('cpu').unsqueeze(0).cpu().numpy()

# onnx 바이너리로 변환된 모델의
# {input_layer_name: input} 형태의 딕셔너리로 초기화
x = {session.get_inputs()[0].name: x}

# 추론
y = session.run(None, x)
```

## 트러블슈팅

- 동적 input을 가지는 모델을 `dynamic_axes` 정의 없이 변환 후 추론을 시도하는 경우:

  당연하지만 변환 이전의 모델에게 집어넣었던대로 넣으려고 하면 더미로 설정했던 input layer shape과 맞지 않다며 에러가 난다.

  → 변환시 `input_names`와 맞추어 `dynamic_axes` 파라미터를 추가로 넣어준다.

  ``` log
  InvalidArgument                           Traceback (most recent call last)
  <ipython-input-6-8a0625091070> in <module>
        8 # compute ONNX Runtime output prediction
        9 ort_inputs = {ort_session.get_inputs()[0].name: to_numpy(x)}
  ---> 10 ort_outs = ort_session.run(None, ort_inputs)
      11 
      12 # compare ONNX Runtime and PyTorch results

  ~/anaconda3/envs/aicr_v13_tensorrt/lib/python3.6/site-packages/onnxruntime/capi/onnxruntime_inference_collection.py in run(self, output_names, input_feed, run_options)
      186             output_names = [output.name for output in self._outputs_meta]
      187         try:
  --> 188             return self._sess.run(output_names, input_feed, run_options)
      189         except C.EPFail as err:
      190             if self._enable_fallback:

  InvalidArgument: [ONNXRuntimeError] : 2 : INVALID_ARGUMENT : Got invalid dimensions for input: inputs for the following indices
  index: 2 Got: 3680 Expected: 960
  index: 3 Got: 2624 Expected: 640
  Please fix either the inputs or the model.
  ```

- 지정된 `opset_version`에 레이어에서 수행하는 연산이 존재하지 않는 경우:

  아래와 같이 특정 operation (`upsample_bilinear2d`) 을 찾을 수 없다고 오류가 발생한다.

  → 변환시 `opset_version`을 변경해본다.
 
  (근데 내 경우 `opset_version`을 10에서 11로 올리니 해결되었는데 `1.5.0`버전의 `torch.onnx.symbolic_opset10.py`와 `torch.onnx.symbolic_opset11.py` 모두 해당 오퍼레이션 타이틀이 코드 내에 명시되어있음. 실제 매핑되는 부분은 다른 부분인듯 하니 확인시에 참고하자)

  ``` log
  RuntimeError                              Traceback (most recent call last)
  <ipython-input-7-483f9e13366c> in <module>
      19     dynamic_axes=dynamic_axes,
      20     export_params=True,
  ---> 21     opset_version=10
      22 )

  ~/anaconda3/envs/aicr_v13/lib/python3.6/site-packages/torch/onnx/__init__.py in export(model, args, f, export_params, verbose, training, input_names, output_names, aten, export_raw_ir, operator_export_type, opset_version, _retain_param_name, do_constant_folding, example_outputs, strip_doc_string, dynamic_axes, keep_initializers_as_inputs, custom_opsets, enable_onnx_checker, use_external_data_format)
      166                         do_constant_folding, example_outputs,
      167                         strip_doc_string, dynamic_axes, keep_initializers_as_inputs,
  --> 168                         custom_opsets, enable_onnx_checker, use_external_data_format)
      169 
      170 

  ~/anaconda3/envs/aicr_v13/lib/python3.6/site-packages/torch/onnx/utils.py in export(model, args, f, export_params, verbose, training, input_names, output_names, aten, export_raw_ir, operator_export_type, opset_version, _retain_param_name, do_constant_folding, example_outputs, strip_doc_string, dynamic_axes, keep_initializers_as_inputs, custom_opsets, enable_onnx_checker, use_external_data_format)
      67             dynamic_axes=dynamic_axes, keep_initializers_as_inputs=keep_initializers_as_inputs,
      68             custom_opsets=custom_opsets, enable_onnx_checker=enable_onnx_checker,
  ---> 69             use_external_data_format=use_external_data_format)
      70 
      71 

  ~/anaconda3/envs/aicr_v13/lib/python3.6/site-packages/torch/onnx/utils.py in _export(model, args, f, export_params, verbose, training, input_names, output_names, operator_export_type, export_type, example_outputs, propagate, opset_version, _retain_param_name, do_constant_folding, strip_doc_string, dynamic_axes, keep_initializers_as_inputs, fixed_batch_size, custom_opsets, add_node_names, enable_onnx_checker, use_external_data_format)
      501                 params_dict, opset_version, dynamic_axes, defer_weight_export,
      502                 operator_export_type, strip_doc_string, val_keep_init_as_ip, custom_opsets,
  --> 503                 val_add_node_names, val_use_external_data_format, model_file_location)
      504         else:
      505             proto, export_map = graph._export_onnx(

  RuntimeError: ONNX export failed: Couldn't export operator aten::upsample_bilinear2d
  ```

- `export_params` 파라미터를 `False`로 주고 변환 후 추론을 시도하는 경우:

  → 오류가 발생하므로 변환시에 가중치를 포함하든지, `InferenceSession`을 초기화 할 때 등 `onnx` 레벨에서 가중치를 로드하는 과정이 선행되어야 할 듯 하다.

  ``` log
  ValueError                                Traceback (most recent call last)
  <ipython-input-6-8a0625091070> in <module>
        8 # compute ONNX Runtime output prediction
        9 ort_inputs = {ort_session.get_inputs()[0].name: to_numpy(x)}
  ---> 10 ort_outs = ort_session.run(None, ort_inputs)
      11 
      12 # compare ONNX Runtime and PyTorch results

  ~/anaconda3/envs/aicr_v13_tensorrt/lib/python3.6/site-packages/onnxruntime/capi/onnxruntime_inference_collection.py in run(self, output_names, input_feed, run_options)
      182         # the graph may have optional inputs used to override initializers. allow for that.
      183         if num_inputs < num_required_inputs:
  --> 184             raise ValueError("Model requires {} inputs. Input Feed contains {}".format(num_required_inputs, num_inputs))
      185         if not output_names:
      186             output_names = [output.name for output in self._outputs_meta]

  ValueError: Model requires 552 inputs. Input Feed contains 1
  ```

