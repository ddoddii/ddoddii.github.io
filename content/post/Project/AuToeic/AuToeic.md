+++
author = "Soeun"
title = "AuToeic 을 소개합니다 !"
date = "2023-05-03"
description = "토익 part.1 문제 자동 출제 모델"
categories = [
    "Project"
]
tags = [
    "AuToeic"
]
image = ""
+++

## AuToeic 등장 배경 

Toeic 시험을 치뤄보신 분이라면, Part .1 의 사진 묘사 문제를 알 것이다. 아래와 같은 이미지가 주어지고, 듣기 평가를 통해 사진에 가장 적합한 설명을 고르는 문제이다.

4가지 선지가 나오는데, 보통 정답1개, 매력적인 오답 1개, 아예 틀린 선지 2개 이렇게 구성된다. 

![img](https://github.com/ddoddii/DSL-23-1-modeling-AuToeic/assets/95014836/f5f765c9-4a3d-4fd2-b6f9-0ac3b3a6c100)

토익을 풀었는데도, 이 문제를 기억하지 못하는 분들도 많을 것이다. 왜냐하면 이 문제는 정말정말 쉽다 !! (그만큼 존재감이 토익 내에서 적다)

우리는 이런 배경을 활용하여, 토익을 출제하는 입장에서 이 사소하지만 꼭 출제해야하는 토익 Part.1 사진 묘사 문제를 원하는 상황만 입력한다면 자동으로 사진과 선지 4개까지 생성해주는 모델을 만들어보기로 했다. AuToeic 이란 '토익'(Toeic) 문제를 '자동'(Auto) 으로 출제해주겠다는 점에서 이름을 지었다. 

연세대학교 Data Science Lab 의 팀원 3명과 같이 진행했으며, 지금부터 우리가 한 프로젝트에 대해 자세히 설명해보겠다. 

## AuToeic 소개 

우리의 모델은 크게 3가지 파트로 구상하였다. 

1. AuToeic 사용자가 문제를 출제하고 싶은 상황 (ex. A bike leaning against the wall) 을 입력하면, 입력한 prompt 를 토대로 흑백 사진을 생성준다. 
2. 사진을 이용하여 상황을 맞게 설명하는 정답선지 1개와, 매력적인 오답 선지 1개, 완전히 틀린 선지 2개를 생성한다. 
3. 우리가 만든 문제를 검증하기 위하여 사진과 선지들 간의 적중률, 즉 정답률을 계산하여 문제와 선지가 올바르게 만들어졌는지를 평가한다. 

아래에서 우리가 어떠한 모델을 썼고, 왜 사용했는지 구체적으로 설명해보겠다. 

## AuToeic 에서 사용한 모델 소개

### 1. Text to Image : Stable Diffusion 1.5 

Stable Diffusion Model 은 사용자가 입력한 텍스트를 토대로 이미지를 만들어주는 모델이다. 
Latent Diffusion이라는 원리로 작동하는데, 이것은 1) 학습한 이미지 데이터셋에 무작위적인 Gaussian noise를 늘려가다가 2) 다시 줄여가면서 처음에 입력받은 텍스트에 상응하는 이미지를 생성하는 과정을 말한다. Stable Diffusion은 지금까지 공개된 데이터셋 중 가장 규모가 큰 LAION-5B 데이터셋으로 훈련되었고, 이미지 생성 모델 중 SOTA(State of the art, 현존하는 최고 수준 성능의 모델)를 달성했다. 더불어 Pixel-space diffusion model보다 메모리와 연상 요구량을 획기적으로 줄였다.

본 프로젝트에서는 Stable Diffusion 1.5 모델을 기반으로 사실적인 이미지들을 만들어주는 [Dreamlike Photoreal 2.0](https://huggingface.co/dreamlike-art/dreamlike-photoreal-2.0) 모델을 사용하였다. 이 모델은 기본 stable diffusion 1.5 모델에 초고화질 (768*768px) 이미지셋을 추가하여 훈련한 모델이다. 토익 Part 1의 사진은 실제 일상과 관련된 상황으로 구성되기 때문에 AuToeic은 이질적이지 않은 사진이 생성해야 한다. 따라서 직접 찍은 사진인지 만들어낸 사진인지 모를 정도의 이미지를 생성할 수 있는 Dreamlike Photoreal 2.0을 활용하였다.

### 2. Image to Text : BLIP 

BLIP은 2022년 공개된 Vision Language Pretraining-model이다. 웹 크롤링으로 수집한, noise가 많은 이미지-텍스트 쌍을 데이터셋으로 학습한 모델인데, 이때 이미지에 상응하는 caption을 bootstrapping하여 노이즈가 많은 웹 데이터를 효과적으로 활용한다.
 
BLIP은 captioner + filter로 이루어져있는데, captioner는 caption을 합성하고, filter는 그것에서 noise를 줄이는 역할을 한다. BLIP 은 3가지 loss function 을 사용하여 학습을 진행하는데, 이것은 기존에 있는 Image to Text model 의 단점들을 보완한다. BLIP 의 가장 중요한 점은 captioner 과 filter 가 caption을 bootstrapping 하면서 성능을 높였고, 더욱 다양한 caption 을 생성한다는 점이다.

### 3. Evaluation : CLIP

OpenAI에서 2021년 1월 5일에 공개한 CLIP(Contrastive Language-Image Pre-training) 모델은 이미지에 가장 적절한 텍스트를 출력해주는 생성모델이다. BLIP과 유사하게도, CLIP은 Pre-training 과정에서 CLIP은 웹 크롤링을 통해 4억 개의 이미지와 연관 텍스트를 추출하여 거대한 데이터셋을 스스로 구축하였다는 점에서 효율적이다. 이미지 수집과 정답 label 생성에 많은 노력이 필요하지 않기 때문이다.

수집한 데이터를 특징 벡터로 변환한 후(Encoder), CLIP은 ‘대조 학습(contrastive learning)’이라는 방법으로 Pre-training 과정을 진행한다. 대조 학습이란 정답인 (이미지, 텍스트) 각 특징 벡터 간의 높은 코사인 유사도와 오답인 (이미지, 텍스트) 각 특징 벡터 간의 의 낮은 코사인 유사도를 모두 구해서 올바른 연결 관계를 학습하는 방법을 일컫는다. 이렇게 학습한 뒤, 새로운 이미지를 입력하고 정답/오답 텍스트를 여러 개 입력하여 정답을 골라내도록 했을 때 CLIP은 매우 높은 확률로 정답 텍스트를 선정해냈다. 많은 실험을 거친 결과, CLIP이 학습 과정에서 배우지 않은 일을 해내는 것(zero-shot learning)에 효과적이라는 사실이 밝혀졌다.

## Model Finetuning

위의 모델들을 우리의 프로젝트에 맞게 Fine-Tuning 하는 과정을 거쳤다. 

- Stable Diffusion 1.5
  - 우리가 실제로 수집한 토익 이미지-정답+오답 500개의 문제들을 사용하였다. 이미지-텍스트 데이터를 사용하여 Dreamlike Photoreal 2.0 을 Fine Tuning 했다.
  - 이미지 수집은 우리가 갖고 있는 토익 문제집의 사진들, 온라인에 공개된 토익 모의고사 사진을 이용했다. 그리고 뒤의 정답지에 있는 정답,오답 선지들을 수집했으며, 만약 선지들이 없다면 우리의 판단하에 선지들을 직접 만들었다. 

- BLIP 
  - 위의 모델과 마찬가지로 우리가 실제로 수집한 토익 이미지-정답+오답 500개의 문제들을 사용하였다. 실제 토익 문제 데이터를 8:2 의 비율로 train, test 으로 나누고, train set 으로 학습을 진행하였고 Test set 으로 평가하였다.
  - 최종적으로 BLIP Model 4개를 사용하였는데, 각각 이미지-정답, 이미지-매력오답, 이미지-완전오답1, 이미지-완전오답2 pair 로 훈련시켰다. 그래서 이미지 하나를 넣으면, 모델 4가지를 사용하여 선지를 하나씩 생성하도록 만들었다. 

- CLIP
  - CLIP 모델은 fine-tuning 하지 않고, pre-train 된 모델 그대로 사용하였다. 

## AuToeic 의 구조적 특징

1. 3가지 모델을 하나의 과정으로 조합했다.

   이전까지 단순히 Text to Image , Image to Text 모델을 사용한 프로젝트는 많았다. 하지만 우리는 그 두가지 모델을 결합하여, Text to Image 모델을 사용하여 문제 사진을 만들고, Toeic Image 로 훈련시킨 4개의 BLIP 모델을 사용하여 정답선지 1개, 매력오답선지 1개, 완전오답선지 2개 를 만들었다. 

2. BLIP 과 CLIP 을 독창적으로 활용했다.
   
    BLIP 모델을 통해 이미지를 captioning을 하는 과정은 이미 다른 프로젝트에서도 많이 활용되고 있다. 대부분의 프로젝트에서 이미지에 대해 옳은 caption을 생성하는 방향으로 BLIP 모델을 학습시킨다. 반면, Autoeic은 의도적으로 적당한(매력적인) 오답의 caption을 생성하도록 유도하는데에도 BLIP의 원리를 활용하였다는 점에서 다른 프로젝트와 구분된다. 정답 선지가 너무 자명하거나 오답 선지가 완벽하게 오답일 경우 문제의 가치가 떨어지기 때문에, 같거나 유사한 소재, 어구, 동사 등을 포함하는 매력적인 오답 선지가 필요하기 때문이다.

    더불어 CLIP의 대조 학습(contrastive learning) 방법을 1개의 사진과 4개의 caption(선지) 간 연관성을 “평가”하는 데에 응용한 것은 AuToeic이 최초로 시도하는 것으로 파악된다. 생성한 사진과 정답/오답 caption 각각의 특징 벡터를 구한 후 계산한 상호 유사도는 caption이 사진을 얼마나 구체적으로 설명하는지에 대한 확률값이다. 따라서 유사도는 수험자들이 각 선지를 선택할 확률로 이해할 수 있으며, 문항의 정확도와 난이도의 평가 지표가 된다. 만약 사진과 유사한 선지가 2개 이상 생성되었다면, 복수정답의 위험이 있는 문제이므로, 각각 사진-선지의 유사도 확률이 하나만 높고, 나머지 3개는 낮도록 조정하는 식으로 자체 문제를 검증하였다. 이렇게 문항을 평가하는 과정은 실제 수험자가 문항을 해결하는 과정과 상당히 유사하므로, 모델이 출제한 문항을 사람이 직접 정성적으로 평가할 필요가 없으며 문제를 자체적으로 검증하는 수단도 자동화 시켰다.
   

## 실제 실행 결과

### **문제1.**

입력 Text : Black and white photo, Some bicycles have been left unattained

생성 이미지: 
![img](https://github.com/ddoddii/DSL-23-1-modeling-AuToeic/assets/95014836/3126721a-a82c-468d-b8cc-4ac88ceb9585)

실제 문제 선지 4개 (정답1개 + 오답 3개)
![img](https://github.com/ddoddii/DSL-23-1-modeling-AuToeic/assets/95014836/3e1c76b5-1695-4af1-9fd5-d2f7a68243bf)

### **문제2.**

입력 Text: black and white photo, a man looking at a computer

생성 이미지:
![img](https://github.com/ddoddii/DSL-23-1-modeling-AuToeic/assets/95014836/8ab697e0-c039-4a43-9c41-0e537a2e4140)

실제 문제 선지 4개 (정답1개 + 오답 3개)
![img](https://github.com/ddoddii/DSL-23-1-modeling-AuToeic/assets/95014836/bd1f1c3f-47a2-4090-af2d-89074389d932)

위의 결과를 보면, 문제1은 하나의 정답 선지만 사진을 적절하게 설명하는 것으로 나오고, 문제2는 다른 선지도 사진을 어느 정도 설명하는 것으로 평가할 수 있다. 이 과정을 통해서 어느 문제가 출제가 잘 되었는지, 다시 한번 검토해볼 수 있다.

## 느낀 점

이 아이디어의 맨 처음 시작은 Kaggle 의 Stable Diffusion 을 이용한 공모전이었다. 하지만 회의를 진행하며 우리는 공모전 보다는 실생활에 좀 더 도움이 되는 프로젝트를 할 수 없을까? 생각하다가 최근에 토익을 보고 온 성균이가 아이디어를 제일 처음 제시해줬다. 학기 중에 모두 바빴는데도 대면회의를 많이 했는데, 매 회의마다 정말 재밌고 , 아이디어가 넘쳤다. 

정말 훌륭한 팀원들을 만나 팀장으로 활동하며 좋은 결과물을 낼 수 있어서 굉장히 애착이 많이 가고 뿌듯한 프로젝트이다. 실제로 학회 내에서 발표를 했을 때 재우가 영국에서 살다와서 직접 선지를 읽어주고 우리가 만든 문제를 풀어보게 하면서 발표를 시작했는데, 좋은 평가를 받았다. 그리고 아이디어가 독창적이라는 이야기를 들어서 "우리 특허 한번 출원해볼까 ?!!?" 라고 해서 학회 지도교수님의 도움을 받아 특허 출원에 도전해보기로 했다. 앞으로 할 수 많은 프로젝트를 할 테지만, 이 프로젝트가 가장 기억에 많이 남을 것 같다.

우리끼리 너무 재밌어서 회의를 핑계삼아 많이도 놀러다녔다.. 개그맨이었던 규원오빠와 아이디어 넘치던 성균이 회의 올때마다 야무지게 해오던 재우까지 정말 최고의 팀이었다 !

![KakaoTalk_Photo_2023-08-03-14-59-08](https://github.com/ddoddii/DSL-23-1-modeling-AuToeic/assets/95014836/2ff61a4c-40ff-4865-833a-3f2a52da887a)


![KakaoTalk_Photo_2023-08-03-14-58-27](https://github.com/ddoddii/DSL-23-1-modeling-AuToeic/assets/95014836/74ea0a95-36ca-4e00-98cc-e587c3d2d0a9)

## 코드와 설명서
- 사용한 코드는 아래 GitHub 에서 볼 수 있습니다.
- Code : [GitHub](https://github.com/ddoddii/DSL-23-1-modeling-AuToeic)