+++
author = "Soeun"
title = "ML Model Deployment"
date = "2023-07-01"
summary = "ML Model 배포하기 - 1"
categories = [
    "DS"
]
tags = [
    "MLOps"
]
image = ""
+++
### Model Deployment 이란 무엇일까?

ML Model 을 deploy(배포) 한다는 것은 우리가 만든 모델을 프로덕션 환경에서 쓸 수 있게 하는 것이다. ML 라이프 사이클에서 가장 마지막 단계에 위치하며, 어쩌면 가장 어려운 단계이다. 

### 왜 Model Deployment 이 어려울까?

- 기존 소프트웨어에서 pain point 들
    - Reliability
    - Reusability
    - Maintainability
    - Flexibility
- ML Model에서 추가된 pain point
    - Reproducibility
        
        Black box 모델이 많은 ML, DL 모델의 특성 상 같은 결과가 나오지 않는 경우가 많다. 따라서 ML 모델을 프로덕션으로 사용할 때, Reproducibility(재현가능성) 은 정말 신경 써야할 지점이다. 
        
- DS 팀은 정말 많은 사람들이 얽혀 있다. data scientists, IT teams, software developers, business professionals 등등..
![img](https://github.com/ddoddii/skills-for-DS/assets/95014836/3d321d7b-54f9-48e3-b21a-7f86c5fb0682)
- 모델을 만든 프로그래밍 언어와, Deploy 단계에서 쓰는 언어가 다를 수 있다. 보통 ML 모델은 Python 으로 작업을 하는데, 백엔드를 위해서는 javascript 이 쓰이는 경우가 있다.

### 그럼에도 왜 Model Deployment 는 중요할까?

- ML 모델을 사용하려면 무조건 deploy 해야 다른 사람들도 내가 만든 ML product 를 쓸 수 있다 !

### Reference
- Udemy, Deployment of Machine Learning Models