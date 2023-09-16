+++
author = "Soeun"
title = "Multimodal 기초"
date = "2023-08-23"
description = "Multimodal 기초"
categories = [
    "DS"
]
tags = [
    "Multimodal"
]
draft = true
image = ""
+++

## Introduction

### Background
- 인간 행동 인식이나 감정 인식 문제에 딥러닝을 적용하고자 하면 ?
  ![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d38c6351-2971-4a43-bb7d-caae1a39d237)
- 그러나 인간 행동 인식이나, 감정 인식 문제에서는 이미지를 잘 분류한다고 해서 성능이 확보되지 않는다.
- 사람을 표현하는 데이터로 단순 이미지가 아니라, 행동, 목소리, 대화, 표정을 모두 사용하면 감정을 분류하기 더 쉽지 않을까 ?
- 그렇다면 모델에 넣은 Input Data 는 다양한 형태 (음성, 사진, 텍스트) 이어야 한다. 이것을 **Multimodal Data** 라고 부른다.
   ![Img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/164640ec-fdfa-4cab-8838-028f4b084ec9)

### Multimodal Deep Learning
- 단일 모달(single modal) 데이터의 한계를 극복하고자 여러 모달 (Multimodal)의 데이터를 사용하여 주어진 문제 (e.g. 감정분류)를 해결하는 모델을 구축하는 방법론
- 모달(modality)라는 것은 **데이터의 형태**를 의미
- Visual, Auditory, Reading, Writing, Kinesthetic 등 다양한 형태가 될 수 있다
![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/56151001-46c8-4d24-ad4f-7a2100a178e1)

- 각 모달에 적합한 딥러닝 구조를 사용하여 특징 벡터를 추출
- 모달을 통합하는 방식에는 대표적으로 (a) feature concatenation , (b) ensemble classifier 두 가지 방법이 존재
- (a) Feature Concatenation
  - Modality 1(e.g text) 는 RNN, Modality 2(e.g image)는 CNN 을 사용하여 Feature vector 를 추출한 후, 벡터 레벨에서 합치는 것
  - 이후 합쳐진 벡터를 이용해서 down stream task 를 수행한다
- (b) Ensemble Classifier
  - 각 Modality 에서 각각 label 까지 도출한 후, ensemble classifier 를 사용하여 최종 label 을 도출하는 형태
  - 최근에는 잘 쓰이지 않는다
  
![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3cdb80ed-4dea-46d0-ac89-e396549cf213)
- 최근 멀티모달 관련 연구 흐름은 (a)방식에서 transformer 계열까지 적용한 구조까지 발전 중
- Multimodal space 에 비슷한 의미를 가진 벡터들을 비슷한 위치에 embedding 하는 것이 목표이다
    ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/796fd86f-2333-41ed-802e-990c4799c5c4)


### Reference
- [DMQA Seminar Multimodal Learning](https://www.youtube.com/watch?v=f4caa0izZBg)