+++
author = "Soeun"
title = "Cloud Service"
date = "2023-07-17"
description = "Cloud Service 개념 알아보기"
categories = [
    "CS"
]
tags = [
    "AWS",
    "GCP"
]
image = ""
+++

## 1. Cloud Service 

- **Cloud Service 개념**
  - 클라우드 서비스란, 인터넷을 통해 컴퓨팅 자원, 데이터 저장, 소프트웨어, 플랫폼 및 기타 IT 관련 서비스를 원격으로 제공하는 것
  - 클라우드 서비스는 필요한 리소스(하드웨어, 소프트웨어, 데이터 저장소 등)을 필요한 만큼 요청하고 제공받는 **온디맨드(on-demand)** 방식으로 작동
  - 사용자는 필요한 시점에 적절한 양의 자원을 신속하게 할당받거나 반환할 수 있음
  - 사용자는 자원의 유연성과 확장성을 활용하여 비용을 절감
  - 온프레미즈(on-premise) : 물리적으로 서버실을 구축해놓고 안 쓸 때도 비용이 나감
  - 온디맨드(on-demand) : 필요할 때마다 사용하는 방식
    ![img](https://github.com/ddoddii/skills-for-DS/assets/95014836/5368fb89-280e-4cd6-aedf-31f53a52c44d)

- **Cloud Service 장점**
  - 서버실을 직접 물리적으로 세팅할 필요 X!
  - 데이터 센터 어딘가에 이미 준비된 서버 사용 (Amazon, Google 과 같은 대기업들이 이미 구축해놓은 거대한 서버실)
  - 서버 세팅 등을 신경쓰지 않고 서비스 운영에만 집중 가능
  - 사용한 만큼 비용을 지불하여 서비스 운영에 효율성이 높아짐 (분단위 , 시간단위 과금)

- **Cloud Service 의 유형**
  - **IaaS** (Infrastructure as a Service)
    - 사용자가 관리 할 수 있는 범위가 가장 넓은 클라우드 컴퓨팅 서비스 = 서버에서 제공하는 범위가 가장 작다
    - 사용자가 서버 OS, 미들웨어, 런타임, 데이터, 어플리케이션까지 직접 구성하고 관리 가능
    - AWS의 EC2 와 Google 의 Compute Engine
  - **PaaS** (Platform as a Service)
    - IaaS 형태의 가상화된 클라우드 위에 사용자가 원하는 서비스를 개발할 수 있도록 개발 환경 (Platform)을 미리 구축
    - IaaS 보다는 관리상의 자유도가 낮다
    - Salseforce의 Heroku , Redhat의 Openshift
  - **SaaS** (Software as a Service)
    - 바로 사용할 수 있는 소프트웨어 자체를 제공하는 것
    - 사용자는 구독료를 지불하고 소프트웨어를 이용
    - Slack, Microsoft 365, Dropbox
    ![img](https://github.com/ddoddii/skills-for-DS/assets/95014836/8a5991c2-3351-48c3-8e97-4a1572527f92)

- Cloud Service 는 어떤 것을 써야 할까? 
  - Top 3 : AWS, Azure, Google Cloud
  - 각각의 장단점이 있다
  - 운영하고자 하는 서비스에 맞게 적절한 클라우드 사업자를 선택해야 한다
  - 실제로는 여러 클라우드 컴퓨팅 업체를 이용하는 경우도 많다