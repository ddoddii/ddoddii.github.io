+++
author = "Soeun"
title = "Multimodal Introduction"
date = "2023-08-24"
description = "CMU Multimodal Machine Learning course, Fall 2022, Lec 1.1"
categories = [
    "DS"
]
tags = [
    "Multimodal"
]
image = ""
+++

## What is Multimodal ?
- Robots, Mobile phone, Vehicles, youtube, wearable...
- 수학적 정의가 아니라 '감각' 에 초점을 맞춘다.
- 인간이 소통하는 방식
  - Verbal (내용언어) = 전달하고자 하는 내용 
  - Vocal (청각언어) = 들리는 언어 (음성,강약,어투,발음)
  - Visual (시각언어) = 보이는 언어 (표정,태도)
- Modality
  - 정의 : Modality refers to the way in which something is expressed or perceived 
  - Modality 인터랙션 과정에서 의사소통을 위해 데이터가 표현되거나 받아들여지는 것이다. 
  - 멀티모달은 여러 가지 형태와 의미로 컴퓨터와 대화하는 환경을 말한다. 
  - Sensor 에서 정보를 받아온다고 하자. 

   ![img](https://github.com/ddoddii/Study-repo/assets/95014836/4027bda3-f000-4907-a8d3-0756bb95c523)
- Multimodal
  - Multimodal : with multiple modalities (dictionary definition)
  - Multimodal : the science of **heterogeneous** and **interconnected** data (research-oriented definition)
- Heterogeneous Modalities
  - Information present in different modalities will often show diverse qualities, structures and representations
  - Abstract modality 는 Homogeneous modality 에 가까울 가능성이 크고, Raw modality 는 Heterogeneous modality 일 가능성이 크다. 
    ![img2](https://github.com/ddoddii/Study-repo/assets/95014836/6261c7cd-8781-44d9-bfb3-f707977ba4ef)
- Dimensions of Heterogeneity
  - Information present in differenct modalities will often show diverse qualities, structures, and representations

## Interconnected Modalities
- Modality connections
  - Modalities are often related and share commonality
  - Statistical : 얼마나 많이 같이 등장하는가 
  - Semantic : 맥락 상에서 얼마나 비슷한가
  ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b9f2964e-33bd-4938-9b28-1a8e2803c0d0)
- Modality Interactions
  - Modality elements often interact during inference
  - 만약 두 modal 의 추론 결과가 다르면 하나의 결과가 다른 결과를 'take-over' 할 수 있다.
  ![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/86b5a082-790b-4350-9c72-98fdf58380fc)

## What is Multimodal Machine Learning?
- Multimodal Machine Learning(ML) is the study of computer algorithms that learn and improve through the use and experience of data from multiple modalities
- Multimodal Artificial Intelligence(AI) studies computer agents able to demonstrate intelligence capabilities such as understanding, reasoning and planning, through multimodal experiences, and data
- Multimodal AI 는 Multimodal ML의 부분집합이다.
    ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0928226f-90b5-4ce6-bd51-d33936cefa06)

## Core multimodal technical challenges

### Challenge 1 : Representation
- 정의 : Learning representations that reflect cross-modal interactions between individual elements, across different modalities
  -> Core building block for most multimodal modeling problems !
- 'I like it' , 행복한 표정, 말투는 Multimodal space 에 유사하게 encode 될 것이다. 
- 이러한 Multimodal vector space 에서의 연산을 잘 다루는 것이 중요하다.
  ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/796fd86f-2333-41ed-802e-990c4799c5c4)

- 이런 Representation 에는 3가지 종류가 있다.
- Fusion : 두 개의 Modality가 있을때, 이 둘을 합쳐서 한 개의 vector space를 갖는 Representation으로 나타내는 것을 의미
-  Coordination : 두 개의 Modality를 합치는 것이 아니라 그들 각각의 space를 갖는 Representation을 구해서 sub space를 만든 후 스팩트럼처럼 조정할 수 있음
-  Fission : 두 개의 Modality 가 있을 때, Modality 의 수보다 더 많은 vector space 에 나타내는 것
  ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/da886fc1-cd98-471d-ae6e-ff85a510a3ed)

### Challenge 2 : Alignment
- 정의 : Identifying and modeling cross-model connections between all elements of multiple modalities, building from the data structure
  -> Most modalities have internal structure with multiple elements
- synchronize 를 말함. 사람이 말할 때 음성과 행동을 동시에 하기 때문에 이 둘을 맞춰주는 것이 필요하다.
- Sub-challenges
  - Connections
  - Aligned Representation
  - Elements
  ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a1243d24-edbf-4b39-9311-2314f3bc1668)

### Challenge 3 : Reasoning
- 정의 : Combining knowledge, usually through multiple inferential steps, exploiting multimodal alignment and problem structure

  ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/36436831-b36c-4be7-aa7a-ba926ad60101)
  ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/93d1b5e3-8dc7-4f2e-bbe3-8391f67fb749)

### Challenge 4 : Generation
- 정의 : Learning a generative process to produce raw modalities that reflects cross-modal interactions, stucture and coherence
- Sub-challenges
  - Summarization
  - Translation
  - Creation
  ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ae605c88-5e15-476b-a1e4-dbb7f2160603)

### Challenge 5 : Transferences
- 정의 : Transfer knowledge between modalities, usually to help the target modality which may be noisy or with limited resources
- Sub-challenges
  - Transfer
  - Co-learning
  ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/19c9b2bc-ca25-452e-bab8-e122d68b340c)

### Challenge 6 : Quantification
- 정의 : Empirical and theoretical study to better understand heterogeneity, cross-modal interactions and the multimodal learning process
- Sub-challenges
  - Heterogeneity
  - Interactions
  - Learning
  ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5c859a5f-377e-48d7-8e7e-e1e7b07d563a)

### Summary
![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b9fcaf69-808f-4c85-8170-2214dc79dd1c)


### Reference
- [CMU, Lecture 1.1 - Introduction (CMU Multimodal Machine Learning course, Fall 2022)](https://www.youtube.com/watch?v=6YsbpYSO_QM)
- https://lsh110600.github.io/deeplearning/2021/01/08/dl-multimodal-1/
