+++
author = "Soeun"
title = "Research vs. Production Environments"
date = "2023-07-03"
description = "ML Model 배포하기 - 3"
categories = [
    "DS"
]
tags = [
    "MLOps"
]
image = ""
+++

### Environment

- 환경이란 소프트웨어 또는 기타 제품이 개발되거나 작동 중인 컴퓨터의 설정 또는 상태이다 .

### Research Environment

- Research Environment(연구 환경)는 데이터 분석 및 머신 러닝 모델 개발에 적합한 도구, 프로그램 및 소프트웨어를 갖춘 환경이다.
- 여기서 우리는  데이터를 탐색하며 ML Model을 발전시킨다.
- 대화형 Interpreter인  Jupyter Notebook 을 많이 사용하며, Numpy, Pandas, scikit-learn 패키지들을 사용한다.

![Untitled](https://github.com/ddoddii/skills-for-DS/assets/95014836/265bfe33-f198-4b5b-a1c7-f1438807553f)


### Production Environment

- Production Environment(실행 환경) 은 실시간으로 작동하는 프로그램이 있고, 여러 배포를 위한 툴들이 함께 사용되는 환경이다.
- 여기서 ML Model 을 실제로 비즈니스 용으로 사용할 수 있다.
- .py 파일로 정리하고, 프로젝트를 포장하기 위해 docker 를 사용한다.

![Untitled](https://github.com/ddoddii/skills-for-DS/assets/95014836/cf45fb0f-1a03-43ee-80da-91dd388a1266)

![Untitled](https://github.com/ddoddii/skills-for-DS/assets/95014836/77f68636-cf35-4f08-a8e0-c6c5006b33ef)


이 사진이 Research Environment 와 Production Environment 의 차이를 정말 잘 보여준다. 

Research Environment 에서는 Historical Data(잘 정제되었고, 바뀌지 않는 데이터) 만 가지고 ML Model을 여러가지 방식으로 훈련시킨다. 그러면 Production Environment 에서는 그 훈련시킨 ML Model 을 가지고, Live Data(실시간으로 들어오는 데이터) 를 이용해 예측을 만들어낸다. 

Live Data 에도 훌륭하게 대응하려면, 우선 Historical Data 가 Live Data 의 특성을 잘 반영해야 하고, ML Model을 여러가지 지표를 이용하여 테스트 해 본 후, Production Environment 로 넘겨주어야 한다.

### Reference
- Udemy, Deployment of Machine Learning Models