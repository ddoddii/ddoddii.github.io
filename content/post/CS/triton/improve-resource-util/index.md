+++
author = "Soeun"
title = "Nvidia Triton Server 에서 리소스 최대한 활용하기 (Throughput, Latency 개선방법)"
date = "2024-01-10"
summary = "삽질을 통해 헤쳐나간 Triton Server 사용기 - 2탄"
categories = [
    "CS"
]
tags = [
    "Triton",
]
slug = "resouce-utilization"
series = ["Triton Inference Server"]
series_order = 2
+++

## Part2. Improving resource utilization

이번 파트에서는 dynamic batching 과concurrent model execution 에 대해 다룹니다. 이 둘은 리소스 활용도를 높여 처리량을 향상시킬 뿐만 아니라 지연을 줄이는 데 사용할 수 있는 중요한 기능입니다. 아래에는 직접 실험 해본 결과를 바탕으로, 실제로 throughput 과 latency 가 얼마만큼 향상되는지 그래프를 첨부했습니다.

### What is Dynamic Batching?

Triton Inference Server 에서 **Dynamic Batching** 은, throughput(처리량)을 극대화하기 위해 하나 이상의 추론 요청을 단일 배치 (배치는 동적으로 생성되어야 함) 로 만드는 것입니다.

**Dynamic Batching** 은 모델의 `config.pbtxt` 에서 선택사항을 지정하여 모델별로 동적 배치를 활성화하고 구성할 수 있습니다. Dynamic Batching은 config.pbtxt 파일에 다음을 추가하여 기본 설정으로 활성화할 수 있습니다:

```
dynamic_batching { }
```

Triton 이 이러한 요청을 지연 없이 처리하는 동안, 사용자는 스케쥴러에 제한된 지연을 할당하기를 선택하여 dynamic batcher 가 처리할 더 많은 추론 요청을 모을 수 있습니다.

```
dynamic_batching {
    max_queue_delay_microseconds: 100
}
```

아래의 샘플 시나리오에 대해 봅시다.

5개의 추론 요청 (A,B,C,D,E) 가 있고, 각각의 batch size 는 4,2,2,6,2 라고 가정합시다. 각 배치는 모델에 의해 처리되려면 `Xms` 의 시간이 필요합니다. 모델이 지원하는 최대 배치 크기는 8입니다. A,C 는 시간 `T = 0` 에 도착하고, B 는 `T = X/3` 에 도착하고, D,E 는 `T = 2* X/3` 에 도착합니다.

![dynamic_batching](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/576dffef-1dc7-4c52-95df-36e56bb23675)

만약 dynamic batching 를 사용하지 않는 경우에는, 모든 요청을 순차적으로 처리하고, 이것은 모든 요청을 처리하는데 `5X ms` 가 걸린다는 것을 의미합니다. (하나의 요청 당 `X ms` 이므로, 5개 요청은 `5X ms` )

dynamic batching 을 사용하면 요청을 GPU 메모리에 더 효율적으로 패킹할 수 있으므로 훨씬 더 빠른 `3X ms` 안에 모든 요청을 처리할 수 있습니다. 더 많은 쿼리들이 적은 사이클 안에 처리될 수 있으므로, 응답의 지연 또한 줄인다. `delay` 를 사용하면, (A,B,C) 가 함께 배치되고, (D,E) 가 함꼐 배치되어 더 좋은 성능을 뽑아낼 수 있습니다.

하지만 주의해야 할 점은 위 시나리오는 아주 이상적인 시나리오라는 점입니다. 실제로는, 모든 실행 요소를 완벽하게 병렬화할 수 없으며, 그 결과 큰 배치의 실생시간은 길어집니다.

그럼에도 위에서 관찰한 것과 같이, dynamic batching 을 사용하면 모델을 서빙하는 동안 latency(지연 시간)과 throughput(처리량) 모두 개선할 수 있습니다. 이 배칭 기능은 stateless model (실행 중간에 상태를 저장하지 않는 모델 - e.g. object detection) 에 대한 솔루션을 제공하는데 중점을 둡니다. Triton 의  [sequence batcher](https://github.com/triton-inference-server/server/blob/main/docs/model_configuration.md#sequence-batcher) 는 stateful model 에 대한 여러 추론 요청을 처리하는데 사용될 수 있습니다.

### Concurrent model execution

Triton Inference Server 는 동일한 모델의 여러 인스턴스를 띄울 수 있으며, 이것은 쿼리들을 병렬적으로 처리할 수 있음을 뜻합니다. Trition 은 사용자의 선호에 따라 동일한 장치(GPU) 혹은 동일한 노드의 다른 장치에 인스턴스를 띄울 수 있습니다. 이러한 사용자 커스텀 가능 기능은 다른 처리량을 가진 모델의 ensemble 을 설정할 때 유용합니다. 더 무거운 모델들은 병렬적 프로세싱을 위해 별도의 GPU 에 생성할 수 있습니다. 이것은 model configuration 의 `instance groups` 옵션을 사용하여 설정할 수 있습니다.

```
instance_group [
  {
    count: 2
    kind: KIND_GPU
    gpus: [ 0, 1 ]
  }
]
```

그럼 아까의 예시를 가지고 병렬 실행을 위해 여러 모델을 추가할 때의 상황에 대해 생각해 봅시다. 이 예시에서는, 하나의 모델이 5개의 쿼리를 처리하는 대신, 2개의 모델이 생성되었습니다 (Model Instances = 2).

![multi_instance](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/98b9458c-4930-42a7-a66a-acf1c329980c)

"no dynamic batching" 케이스의 경우, 각 모델들에게 쿼리가 동일하게 할당됩니다. 사용자는 특정 인스턴스 그룹에 우선순위를 부과할 수 도 있습니다.

dynamic batching 이 있는 multiple instances 의 경우, 다음과 같은 상황이 발생합니다. 아까 B 는 `T = X/3` 에 도착한다고 했습니다. 따라서 약간의 지연 후에 요청이 도착하므로, B 는 두번째 인스턴스를 사용해 처리할 수 있습니다. 그러면 X ms 후, A,C 를 모두 처리한 첫번째 인스턴스는 바로 D,E 를 처리할 수 있습니다.

만약 `X/2 ms` 만큼의 delay 가 허용이 된다면, 인스턴스 1은 `T = X/2` 일 때 A,B,C 를 모두 처리하느라 최대 배치 크기 (8) 가 꽉 찼습니다. 인스턴스 2는 그러면 지연 없이 바로 D,E 를 처리할 수 있습니다.

여기서 중요한 점은  Triton Inference Server가 보다 효율적인 배치 생성과 관련된 정책과 관련하여 유연성을 제공함으로써 리소스 활용도를 향상시켜 지연 시간을 줄이고 처리량을 향상시킨다는 것입니다.

### Demo

Part1 의 예시를 사용하여 dynamic batching 과 concurrent model 의 예시를 보여줍니다.

```sh
docker run -it --gpus all -v ${PWD}:/scratch nvcr.io/nvidia/pytorch:<yy.mm>-py3
cd /scratch
wget https://www.dropbox.com/sh/j3xmli4di1zuv3s/AABzCC1KGbIRe2wRwa3diWKwa/None-ResNet-None-CTC.pth
```

Part1 에서 처럼, `utils` 내의 파일을 사용해서 모델들을 `.onnx` 로 export 하면 됩니다.

현재 `config.pbtxt` 에는 아래와 같이 dynamic_batching { } 과 instance_group 를 추가할 수 있습니다. `instance_group` 에서는 사용자는 크게 2가지를 조정할 수 있습니다. 첫번째로, 각 GPU 에 배치될 각 인스턴스 수입니다. 아래의 예시에서는 각 GPU 당 2개의 인스턴스를 배치합니다. 두번째로는 이 그룹의 타켓 GPU 는 `gpus: [ <device number>, ... <device number> ]` 를 사용해서 지정할 수 있습니다.

`dynamic_batching {}` 을 추가하면, dynamic batching 하는 것을 활성화시킬 수 있습니다. 사용자는 `preferred_batch_size` and `max_queue_delay_microseconds` 를 추가하여 사용 케이스에 맞게 세부적으로 조정할 수 있습니다.

```text
dynamic_batching { }

instance_group [
    {
      count: 2
      kind: KIND_GPU
    }
]
```

```sh
docker run --gpus=all -it --shm-size=256m --rm -p8000:8000 -p8001:8001 -p8002:8002 -v ${PWD}:/workspace/ -v ${PWD}/model_repository:/models nvcr.io/nvidia/tritonserver:23.09-py3 bash
```

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3830099c-8a84-43eb-a703-ce953165bb9d">

### Measuring Performance

- **실험 환경**
  - AWS EC2
  - AMI name :  Deep Learning OSS Nvidia Driver AMI GPU PyTorch 2.0.1 (Ubuntu 20.04) 20240116
  - Instance type : g4dn.xlarge
  - GPU : Tesla T4 (1개)

1번째 터미널에서는 triton server 실행했습니다.

```sh
tritonserver --model-repository=/models
```

2번째 터미널에서 아래와 같은 명령어를 실행했습니다. (performance 측정하기 위한 요청 보내는 용도)

```sh
docker run -it --net=host -v ${PWD}:/workspace/ nvcr.io/nvidia/tritonserver:23.09-py3-sdk bash
```

3번째 터미널에서는 GPU 사용량을 체크했습니다.

```sh
watch -n0.1 nvidia-smi
```

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7261b211-8d25-41fb-b7d6-0e6548269d54">

#### 1. No Dynamic Batching, Single model instance

- `config.pbtxt`

```
name: "text_recognition"
backend: "onnxruntime"
max_batch_size : 8
input [
  {
    name: "input.1"
    data_type: TYPE_FP32
    dims: [ 1, 32, 100 ]
  }
]
output [
  {
    name: "308"
    data_type: TYPE_FP32
    dims: [ 26, 37 ]
  }
]
```

- docker 명령어 : 인스턴스가 1개이므로, gpu 1개 사용

```sh
docker run --gpus=1 -it --shm-size=256m --rm -p8000:8000 -p8001:8001 -p8002:8002 -v ${PWD}:/workspace/ -v ${PWD}/model_repository:/models nvcr.io/nvidia/tritonserver:23.09-py3 bash
```

- 실행 결과

```sh
perf_analyzer -m text_recognition -b 2 --shape input.1:1,32,100 --concurrency-range 2:12:2 --percentile=9
```

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1a92e074-5c84-492c-849b-0bcdd8ffd4b1">

#### 2. Dynamic Batching, Single model instance

- `config.pbtxt `

```
== 생략 ==

dynamic_batching { }
```

- docker 명령어 : 인스턴스가 1개이므로, gpu 1개 사용

```sh
docker run --gpus=1 -it --shm-size=256m --rm -p8000:8000 -p8001:8001 -p8002:8002 -v ${PWD}:/workspace/ -v ${PWD}/model_repository:/models nvcr.io/nvidia/tritonserver:23.09-py3 bash
```

- 실행 결과

```sh
perf_analyzer -m text_recognition -b 2 --shape input.1:1,32,100 --concurrency-range 2:12:2 --percentile=9
```

<img width="564" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/dd4d724a-5287-43d2-8629-66418823ab92">

Dynamic batching 만 추가했는데도 throughput 은 크게 증가하고 (concurrency = 12 일때 2배 차이에 근접한다) , latency 도 크게 감소하였음을 확인할 수 있다.

#### 3. Dynamic Batching, Multiple model instance

- `config.pbtxt`

```
== 생략 ==

dynamic_batching { }

instance_group [
    {
      count: 2
      kind: KIND_GPU
    }
]
```

- docker 명령어

```sh
docker run --gpus=all -it --shm-size=256m --rm -p8000:8000 -p8001:8001 -p8002:8002 -v ${PWD}:/workspace/ -v ${PWD}/model_repository:/models nvcr.io/nvidia/tritonserver:23.09-py3 bash
```

- 실행 결과

```sh
perf_analyzer -m text_recognition -b 2 --shape input.1:1,32,100 --concurrency-range 2:16:2 --percentile=95
```

<img width="569" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ffadc205-31b1-4a4c-ad47-e0ce9dd2a1e2">

#### 3-1 . Dynamic Batching with queue_delay, Multiple model instance

지연 시간을 추가해보았습니다.

- `config.pbtxt`

```
dynamic_batching {max_queue_delay_microseconds : 10 }

instance_group [
    {
      count: 2
      kind: KIND_GPU
    }
]
```

- 실험 결과

max_queue_delay_microseconds 를 10으로 설정했는데, max_queue_delay_microseconds 가 없을 때보다 오히려 살짝 느려진 결과가 나왔습니다. 이러한 파라미터는 모델의 요청량, 모델의 구조에 따라 실험해가며 조정하는 편이 좋아 보입니다.

<img width="567" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a51c3b33-6c3a-43d3-a716-a379e539c757">

### 성능 비교 결과 요약

- Throughput 비교

![through](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8181b6c6-144a-4a71-8b84-2d3e26897ad0)

- Latency 비교

![latency](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b7087153-8369-4666-ab74-4070f3970e08)

정말 No Dynamic Batching, Single model instance 는 throuhput, latency 모두 최악인 것을 확인할 수 있다.

## Reference

- https://github.com/triton-inference-server/tutorials
