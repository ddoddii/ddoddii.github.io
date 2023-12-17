+++
author = "Soeun"
title = "Building Reproducible ML Pipelines"
date = "2023-07-04"
summary = "ML Model 배포하기 - 4"
categories = [
    "DS"
]
tags = [
    "MLOps"
]
image = ""
+++

### Reproducibility

![Untitled](https://github.com/ddoddii/skills-for-DS/assets/95014836/6300a20e-6272-4b62-81ec-74eefda977e7)

- **Reproducibility** : ML model 을 정확하게 재현하는 것. 같은 데이터를 넣었을 때, 같은 결과가 나오는 것
- 만약 이 Reproducibility 가 만족이 안되면, 여러 문제가 발생한다. 금전적인 입장에서는, Research Environment 에 투자하여 최상의 모델을 만들었는데, Production Environment에서 재현이 불가능하면 막대한 손실이 발생한다. 금전적인 손실 뿐만 아니라 인력과 시간의 낭비도 발생한다.  
또한, 과거의 결과들을 재현할 수 있어야 기존의 모델과 비교가 가능해진다. 기존의 모델보다 낫다는 확신이 있어야 비용을 투입하여 모델을 교체하는 작업을 진행하는데, 결과를 재현할 때마다 값이 다르게 나온다면 기존의 모델보다 나은지 확신할 수 없다.

### ML Pipeline

![Untitled](https://github.com/ddoddii/skills-for-DS/assets/95014836/c1308a41-4b67-4a35-9890-6bbf873fe037)


각 파이프라인마다 Reproducibility 를 체크해야 한다. 

- **Gathering Data**
    - 재현하기 가장 어려운 단계이다
    - 문제
        - Training dataset 은 항상 업데이트 된다.
        - SQL을 사용하여 데이터를 load 할 때 순서가 바뀐다.
    - 해결방법
        - 항상 원본 데이터는 보존하고, copy 를 만들어서 작업하기
        - Train dataset 의 스냅샷을 남기기 → 가장 간단하지만, 큰 데이터에는 적합하지 않다.
        - 데이터 소스를 정확한 timestamp 를 이용해서 디자인하기 → 가장 이상적인 솔루션이지만, 데이터 소스를 다시 디자인 하는 데에는 노력이 많이 든다.
- **Feature Creation**
    - 문제
        - 결측치를 랜덤한 값 또는 평균값으로 대체했을 때
        - 피처들을 뽑을 때 복잡한 식을 썼을 때
    - 해결방법
        - Version control 를 이용해 feature 가 어떻게 만들어졌는지 추적한다.
        - Feature Engineering 할 때 사용된 많은 파라미터들은 train data에 의존한다. → 따라서 Data 가 Reproducible 한 것이 굉장히 중요하다
- **Model Training**
    - 문제
        - 일부 ML Model 은 train 할때 Randomness에 의존한다. (ex. Neural Nets 에서 가중치 초기화할때, Dropouts, mini-batches shuffling, traing/testing/validation split)
        - ML Model 은 array를 사용한다. 따라서 데이터를 올바른 순서대로 주는 것이 중요하다.
    - 해결방법
        - 피처들의 순서를 기록해놓기
        - hyperparameters 들을 기록해놓기
        - Randomness 를 요구하는 모델들에 대해서는 seed 를 설정해놓기
        - 최종 모델이 여러 모델들이 합쳐진 구조라면, 그 앙상블의 구조를 기록해놓기
- **Model Deployment**
    - 문제
        - 우리가 train 할 때 사용한 피처가 Live data에는 없는 경우가 있다
        - 다양한 프로그래밍 언어들을 사용했을 수 있다
        - 사람마다 실행하는 환경이 다르다
    - 해결방법
        - 소프트웨어 버전이 정확하게 일치해야 한다 → 도커를 이용해서 패키징하자
        - 환경이 다른 문제 → 도커, 가상환경 등을 이용해서 맞추기


### Reference
- Udemy, Deployment of Machine Learning Models

- https://trainindata.medium.com/how-to-build-and-deploy-a-reproducible-machine-learning-pipeline-20119c0ab941
- https://www.rctatman.com/files/Tatman_2018_ReproducibleML.pdf