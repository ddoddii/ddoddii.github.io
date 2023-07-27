+++
author = "Soeun"
title = "Deployment of ML Pipelines"
date = "2023-07-02"
description = "ML Model 배포하기 - 2"
categories = [
    "DS"
]
tags = [
    "MLOps"
]
image = ""
+++

### Machine Learning Models

우리가 ML Model 을 만들 때를 생각해보자. 

우리는 데이터베이스나, API 를 이용해서 데이터를 수집한다. 수집한 데이터를 토대로 regression , classification 문제인지 정의하고, 적절한 모델을 골라서 데이터를 이용해서 모델을 훈련시킨다. 

그렇게 훈련시킨 모델을 가지고, 새로운 데이터에 대해 Prediction 을 한다. 

![Untitled](https://github.com/ddoddii/skills-for-DS/assets/95014836/76b3ca76-bc9d-454d-b168-33f74b6a681c)

### Data Format and Quality

하지만 우리가 얻는 데이터의 형식과 질은 정말 천차만별이다. 

인터넷에서 크롤링을 하는 경우, html 문자와 의미없는 문장(lorem ipsum..) 이 포함되어 있을 수 있다. Column 별로 결측치가 있을 수도 있고, 하나의 Label 에 너무 많은 데이터가 몰려 있을 수 있다. Outlier 들이 있어서 모델이 영향을 받을 수 도 있다. 

데이터의 형식도 시계열 데이터, 텍스트 데이터, 이미지 데이터 등 매우 다양하다. 모델이 해석 할 수 있도록 embedding 값으로 바꾸어주는 작업도 진행해야 한다. 

따라서 우리는 이러한 데이터를 이용해 모델을 훈련시키기 위해서는, 

- Transform Variables
- Extract Features
- Create New Features

등의 작업을 수행해야 한다. 

> **“Garbage in, Garbage out”**

ML 의 정석과도 같은 말이다. 

아무리 모델이 훌륭해도 쓰레기 데이터를 넣으면, 쓰레기 결과물이 나온다. 

데이터 형식과 질을 잘 보고, 그것에 맞게 전처리를 수행해야 한다. 

### Machine Learning Pipeline

![Untitled](https://github.com/ddoddii/skills-for-DS/assets/95014836/1a45221b-8943-4248-be9a-5979595aa2b9)

크게 본 ML Pipeline 은 위와 같다. 전처리 과정이 포함이 되었는데, 내 생각에는 전체 파이프라인 중 가장 중요한 과정이다.

### Reference
- Udemy, Deployment of Machine Learning Models