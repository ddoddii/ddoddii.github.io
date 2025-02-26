---
title: "Nsight Systems Tutorial"
date: 2025-02-15
draft: false
summary : "nsight systems 으로 하는 프로파일링"
tags: ["nsight","profiling"]
categories : ["Tools"]
slug : "nsight-systems-tutorial"
toc : true
katex : true
markup: 'mmark'
---

{{< katex >}}

## Nsight Systems 란?

NVIDIA Nsight Systems는 성능 측정 툴로, 어플리케이션의 알고리즘을 시각화하고, 최적화할 부분을 제시해주며, 어떠한 크기의 CPU 혹은 GPU에 스케일되며, 큰 서버에서부터 작은 칩(systems-on-a-chip)까지 이용할 수 있다. 

1. 시스템 프로파일링

애플리케이션 최적화를 완벽하게 수행하려면 하드웨어 간 상호 작용을 깊이 분석하여 최대한의 병렬성을 확보해야 한다. Nsight Systems는 객관적인 시스템 전체의 활동 데이터를 통합된 타임라인에서 시각화하여 애플리케이션 개발자가 상관관계, 의존성, 활동, 병목 현상 및 리소스 할당을 조사할 수 있도록 한다. 이를 통해 하드웨어 구성 요소가 조화롭게 작동하는지 확인할 수 있다.

2. 성능 측정

Nsight Systems는 낮은 오버헤드로 성능을 분석하여 최적화를 위해 활용되는 숨겨진 이벤트와 지표를 시각화한다. 이를 통해 CPU 병렬화 및 코어 활용률, GPU 스트리밍 멀티프로세서(SM) 최적화, 시스템 작업 부하, CUDA® 라이브러리 추적, 네트워크 통신, 운영체제(OS) 상호 작용 등을 분석할 수 있다.

3. 플랫폼 간 스케일링

Nsight Systems는 NVIDIA 플랫폼에서 애플리케이션을 개발하기 위한 범용 도구이다. 온프레미스 환경뿐만 아니라 클라우드에서도 활용할 수 있으며, NVIDIA DGX™부터 NVIDIA RTX™ 워크스테이션, 자동차용 NVIDIA DRIVE®, 엣지 AI 및 로보틱스를 위한 NVIDIA Jetson™까지 다양한 NVIDIA 플랫폼에서 확장할 수 있다. Nsight Systems는 AI, 고성능 컴퓨팅(HPC), 프로페셔널 비주얼라이제이션, 게임 애플리케이션을 최적화하는 데 유용한 인사이트를 제공한다.


### Nsight Tools Ecosystem

![Image](https://github.com/user-attachments/assets/3d99091d-7998-4522-9b63-b1c842ec9e74)

Nsight 툴에는 **Nsight Systems** 과 **Nsight Compute** 가 있다. Nsight Systems 는 전 시스템의 하드웨어와 소프트웨어 활동 내역을 포괄적으로 보여준다. Nsight Compute 는 조금 더 디테일한 CUDA Kernel 성능 분석과 소스 코드와의 관계까지 보여준다. 


## Nsight Systems 사용방법

Nsight Systems 를 사용하면 타임라인 뷰를 통해서 어느 지점에서 성능 이슈가 생기는지 알 수 있으며, 어떤 최적화를 적용해야 하는지도 알아낼 수 있다. (이게 바로 엔지니어의 존재 이유 아닐까요?) 성능이 안 나오는 이유는 여러 가지가 있을 수 있다. GPU 상에서 동작하는 CUDA Kernel이 너무 오래 걸릴 수 도 있으며, CPU 상에 돌아가고 있는 프로세스가 병목 현상의 원인이 될 수 도 있다. 또한 Disk IO 일 수 도 있으며, PCI bus 상 CPU-GPU 복사가 오래 걸릴 수 도 있다. Nsight Systems는 이러한 원인을 찾아낼 수 있게 해주는 툴이다. 

### Nsight Systems 다운받기

[해당 페이지](https://developer.nvidia.com/nsight-systems/get-started) 에서 본인의 운영체제에 맞는 Nsigt Systems 를 다운 받을 수 있다. 






## Reference
- [NVIDIA Nsight Systems user guide](https://docs.nvidia.com/nsight-systems/UserGuide/index.html)
- [CUDA Developer Tools Tutorial](https://youtube.com/playlist?list=PL5B692fm6--ukF8S7ul5NmceZhXLRv_lR&si=Mq6O5MDAR3ySbu0r)