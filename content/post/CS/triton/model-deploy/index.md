+++
author = "Soeun"
title = "Nvidia Triton Server 에서 모델 배포하기"
date = "2023-08-20"
summary = "삽질을 통해 헤쳐나간 Triton Server 사용기 - 1탄"
categories = [
    "CS"
]
tags = [
    "Triton",
]
slug = "model-deploy"
series = ["Triton Inference Server"]
series_order = 1
+++

## Part1. model deployment

우선, https://github.com/triton-inference-server/tutorials/tree/main 를 git clone 해야 합니다. (용량을 줄이기 위해 main 브랜치만 clone 했습니다.)

저는 Nvidia Driver 가 없는 맥북을 사용중이므로, AWS 상에서 GPU 가 있는 EC2 인스턴스를 사용했습니다. 이때, Tutorial 에서는 `nvcr.io/nvidia/tritonserver:<xx.yy>` 로 버전을 직접 고를 수 있는데, `nvidia-smi` 로 본인의 cuda 버전을 확인한 후 호환 되는 triton server 버전을 사용해야 합니다. (이것 때문에 3시간 동안 삽질했습니다..)

```sh
docker run --gpus=all -it --shm-size=256m --rm -p8000:8000 -p8001:8001 -p8002:8002 -v $(pwd)/model_repository:/models nvcr.io/nvidia/tritonserver:23.09-py3
```

저는 nvidia-smi 의 cuda 버전 (12.2) 와 호환되는 tritonserver 의 23.09 버전을 사용했습니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cb5d3056-73cb-45c6-a7b5-47b443d5dce5)

튜토리얼에서는 [EAST](https://arxiv.org/pdf/1704.03155v2.pdf) model 을 사용하여, 사진 속 문자를 인식하는 모델을 사용합니다.

EC2 상, 혹은 본인의 컴퓨터에 GPU 가 있다면 로컬에서 git clone 을 한 후

```sh
cd Conceptual_Guide/Part_1-model_deployment
```

### Model 1 : Text Detection

OpenCV's EAST model 을 다운로드하고 unzip 합니다.

```sh
wget https://www.dropbox.com/s/r2ingd0l3zt8hxs/frozen_east_text_detection.tar.gz
tar -xvf frozen_east_text_detection.tar.gz
```

tf2onnx 를 설치한 후 onnx 모델로 export 해줍니다.

```sh
pip install -U tf2onnx
python -m tf2onnx.convert --input frozen_east_text_detection.pb --inputs "input_images:0" --outputs "feature_fusion/Conv_7/Sigmoid:0","feature_fusion/concat_3:0" --output detection.onnx
```

### Model 2 : Text Recognition

Text Recognition model weights 를 다운로드 합니다.

```
wget https://www.dropbox.com/sh/j3xmli4di1zuv3s/AABzCC1KGbIRe2wRwa3diWKwa/None-ResNet-None-CTC.pth
```

아래의 script 를 복사한 후, convert.py 파일을 만들었습니다. 그 후 `python convert.py` 를 실행하면 onnx 모델 파일이 생성됩니다.

```python
import torch
from utils.model import STRModel

# Create PyTorch Model Object
model = STRModel(input_channels=1, output_channels=512, num_classes=37)

# Load model weights from external file
state = torch.load("None-ResNet-None-CTC.pth")
state = {key.replace("module.", ""): value for key, value in state.items()}
model.load_state_dict(state)

# Create ONNX file by tracing model
trace_input = torch.randn(1, 1, 32, 100)
torch.onnx.export(model, trace_input, "str.onnx", verbose=True)
```

모델 레포 구조에 특별한 규칙이 있습니다. 아래와 같은 구조로 레포가 되어야 Triton Inference Server 가 인식합니다.

```
# Example repository structure
<model-repository>/
  <model-name>/
    [config.pbtxt]
    [<output-labels-file> ...]
    <version>/
      <model-definition-file>
    <version>/
      <model-definition-file>
    ...
  <model-name>/
    [config.pbtxt]
    [<output-labels-file> ...]
    <version>/
      <model-definition-file>
    <version>/
      <model-definition-file>
    ...
  ...
```

- `model-name` : 모델의 식별 이름
- `config.pbtxt` : 각 모델에 대해 사용자는 모델 구성을 정의할 수 있습니다. 이 구성은 최소한 모델 입력과 출력의 백엔드, 이름, 모양 및 데이터 유형을 정의해야 합니다. 일반적인 백엔드의 대부분에서 이 구성 파일은 기본값으로 자동 생성됩니다. 구성 파일의 전체 사양은 model_config protobuf 정의에서 찾을 수 있습니다.
- `version` : 동일한 모델에 대해 여러 버전을 정의하고, 나눌 수 있습니다.

튜토리얼에서는 아래의 명령어를 입력하여 위와 같은 구조를 만들 수 있습니다.

```sh
mkdir -p model_repository/text_detection/1
mv detection.onnx model_repository/text_detection/1/model.onnx

mkdir -p model_repository/text_recognition/1
mv str.onnx model_repository/text_recognition/1/model.onnx
```

그렇다면 우리의 레포는 아래와 같이 생겨야 합니다.

```
# Expected folder layout
model_repository/
├── text_detection
│   ├── 1
│   │   └── model.onnx
│   └── config.pbtxt
└── text_recognition
    ├── 1
    │   └── model.onnx
    └── config.pbtxt
```

### Launching the server

docker 를 사용하여 서버를 런칭합니다. 저는 23.09 버전을 사용했습니다.

```sh
# Replace the yy.mm in the image name with the release year and month
# of the Triton version needed, eg. 22.08

docker run --gpus=all -it --shm-size=256m --rm -p8000:8000 -p8001:8001 -p8002:8002 -v $(pwd)/model_repository:/models nvcr.io/nvidia/tritonserver:<yy.mm>-py3
```

그 다음, 아래와 같은 창이 뜨면 성공입니다 👍

<img width="700" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/44b50d9f-18c7-4f3b-b8af-e28a40880d05">

컨테이너 안에 들어왔으면, triton server 를 런칭합니다.

```sh
tritonserver --model-repository=/models
```

성공적이라면, 아래와 같이 모델들이 모두 READY 상태여야 합니다.
![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/32eecadb-6963-497f-96f8-6723bdc193da)

이제 서버를 런칭했으니, 요청을 보낼 client 쪽을 만들어 봅시다.

### Building a client application

Triton Inference Server 에 메시지를 보낼 수 있습니다. Triton Inference Server 와 상호 작용하는 방법은 세 가지가 있습니다:

- HTTP(S) API
- gRPC API
- Native C API

이 튜토리얼에서는 `client.py` 파일을 사용하여 HTTP API 를 이용해 소통해봅시다.  `tritonclient` 라이브러리는 HTTP 요청을 보낼 수 있게 해줍니다.

서버와 다른 터미널 창을 열고, 접속한 다음 아래를 입력합시다.

```sh
pip install tritonclient[http] opencv-python-headless
python client.py
```

우선 img1.jpg 가 처리될 것입니다. 저는 여기에 실험을 위해 go 가 적혀진 표지판 사진도 추론에 이용해 보았습니다. 처음 요청할 때는 좀 시간이 걸리지만, 한번 연결을 해놓으면 추론 속도가 굉장히 빠른 것을 확인할 수 있습니다.

- 첫번째 요청 (go 표지판 이미지)

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/87ef5e6e-0678-4a78-ab58-5557b9c0bc19)

- 이후 요청 (stop sign 이미지)

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8e351ba8-b3b5-4ee8-9049-028530497c0a)

무려 0.22 초 만에 사진을 인식하고, text_detection과 text_recognition 작업을 모두 해냈습니다.

## Reference

- https://github.com/triton-inference-server/tutorials
