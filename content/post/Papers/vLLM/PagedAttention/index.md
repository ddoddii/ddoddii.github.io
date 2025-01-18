---
title: "[Review] - Efficient Memory Management for Large Language  Model Serving with PagedAttention"
date: 2025-01-15
draft: false
summary : "vLLM Paper Review"
tags: ["vLLM","PagedAttention"]
categories : ["Papers"]
slug : "vllm-paper-review"
toc : true
katex : true
markup: 'mmark'
---

{{< katex >}}

## 1. Introduction

GPT와 같은 LLM이 다양하게 사용되면서, 우리 삶의 양상도 변했습니다. 하지만 LLM을 서빙하고 호스팅하는 것은 비싸며 많은 GPU를 필요로 합니다. 따라서 throughput(처리량)을 증가시키고, 요청 당 비용을 감소시키는 것이 *LLM serving system* 에서 매우 중요해졌습니다.

LLM 의 중심에는 **Transformer model**이 있습니다. Transformer는 input(prompt)과 이전 토큰 결과물을 기반으로 한번에 한개의 토큰을 생성합니다. 이 과정은 termination token(EOS token)을 생성할 때까지 반복됩니다. 이 과정은 워크로드를 *memory-bound* 하게 만드는데, GPU 를 전부 활용하지 못하게 제한합니다. 

Throughput 을 증가시키기 위해서는 여러 개의 요청을 한번에 처리하는 **batching** 을 하면 됩니다. 그렇지만 배치로 처리하기 위해서는 각 요청이 사용하는 메모리 공간은 효율적으로 관리되어야 합니다. 

![image](https://github.com/user-attachments/assets/082aac0d-51f0-4a21-969f-d15d803a99ff)

Fig. 1 에서는 13B-paramater LLM 이 40GB RAM의 Nvidia A100 GPU 상에서의 메모리 사용량을 나타냅니다. 65%는 모델 가중치에 할당되고, 30%는 요청의 다이나믹한 상태를 저장하기 위해 사용됩니다. Transformer 에서는 이 상태는 attention 메커니즘과 관련된 key & value 텐서로  이루어집니다. 이것을 *KV cache* 라고도 합니다. 나머지 메모리는 activations 과 같은 다른 데이터를 저장하기 위해 사용됩니다. 이때 모델 가중치는 고정되어 있고, 나머지 데이터는 작은 비중을 차지하기 때문에 KV cache 를 잘 다루는 것이 최대 배치 사이즈를 결정하는데 중요한 역할을 합니다.

기존의 서빙 시스템(Tensorflow-serving, ORCA)는 KV cache 메모리를 효율적으로 관리하지 않고 있습니다. 왜냐하면 이 시스템은 KV cache 를 연속적인 메모리 공간에 저장하고 있습니다. 글나 KV cache 는 독특한 특징이 있는데 - 모델이 새로운 토큰을 생성함에 따라 동적으로 커지고 줄어들기도 하며, 사전에 라이프 사이클과 길이를 알 수 없습니다. 이 특징은 기존 시스템의 접근을 매우 비효율적으로 만듭니다.

첫번째로, 기존 시스템은 <span style="background-color: #FBF595">**내부와 외부 메모리 단편화**</span>가 생깁니다. 요청의 KV cache 를 연속적인 메모리 공간에 저장하려면, 요청의 최대 길이(e.g. 2048 token)만큼 메모리 공간을 미리 할당합니다. 실제 요청의 길이는 최대 길이보다 훨씬 짧을 수 도 있기 때문에 메모리 내부 단편화가 생깁니다. 만약에 길이를 사전에 안다고 해도 사전에 메모리 공간을 할당하는 것은 꽤나 비효율적인데, 왜냐하면 요청의 라이프사이클 내내 메모리 청크가 할당되어 있기 때문에 짧은 요청들은 다른 요청의 현재 청크 내 현재 사용되지 않고 있는 부분들을 사용할 수 없습니다. 추가적으로, 외부 메모리 단편화도 생길 수 있는데, 왜냐면 사전에 할당되는 메모리 공간의 크기가 요청마다 다를 수 있기 때문입니다. 실제로 저자들이 실험한 결과 기존 시스템에서는 KV cache memory의 20.4% - 38.2%만 실제 토큰을 저장하기 위해 사용됩니다(Fig.2).

![image](https://github.com/user-attachments/assets/97a264f1-b5f8-4afc-b357-0afd0473051f)

두번째로, <span style="background-color: #FBF595">**기존 시스템은 메모리 공유를 할 수 없습니다.**</span> LLM 서비스들은 대게 요청 당 여러 개의 아웃풋을 생성하는 최신의 decoding algorithm(parallel sampling, beam search) 를 사용합니다. 이런 시나리오 하에서는 각 요청은 여러 개의 시퀀스들로 이루어져 있는데 시퀀스 끼리는 부분적으로 KV cache 를 공유할 수 있습니다. 하지만 기존 시스템에서는 KV cache 가 각자의 연속적인 메모리 공간에 저장되기 때문에 이러한 메모리 공유가 불가능합니다.

이러한 한계점을 해결하기 위해서 저자들은 <span style="background-color: #FBF595">**PagedAttention**</span> 을 제안합니다. PagedAttention 은 OS의 메모리 단편화의 해결책인 **가상메모리와 페이징** 개념에서 영감을 받은 알고리즘입니다. PagedAttention은 요청의 KV cache 를 블럭 단위로 나누는데, 각 블럭은 고정된 수의 토큰의 attention key 와 value 를 저장하고 있습니다. PagedAttention에서는 KV cache 가 연속적인 공간에 저장될 필요가 없습니다. 따라서 OS의 가상메모리처럼, KV cache 를 더욱 유연하게 관리할 수 있습니다 - 마치 블럭은 페이지, 토큰은 바이트, 요청은 프로세스로 생각하면 됩니다. 이 디자인은 작은 블럭을 사용하고 필요할 때 이들을 할당함으로써 내부 단편화 문제를 해결합니다. 게다가 모든 블럭이 같은 사이즈이기 때문에 외부 단편화 문제를 해결합니다. 마지막으로 같은 요청 내의 다른 시퀀스들이 블럭을 접근하는게 가능해지면서 블럭 단위로 메모리 공유가 가능하게끔 합니다.

이 논문에서는 PagedAttention을 기반으로 한 vLLM 을 소개합니다. vLLM은 KV cache 메모리의 낭비를 거의 0으로 만듭니다.


## 2. Background

### 2.1 Transformer-Based Large Language Models

언어 모델링의 태스크는 결국 토큰의 리스트의 확률을 모델링하는 것입니다. 언어는 자연적으로 순서가 있기 때문에, 전체 시퀀스의 결합 확률(joint probability)을 조건부 확률의 곱으로 분해하여 계산하는 것이 일반적입니다.

$$P(x) = P(x_{1}) \cdot P(x_{2} | x_{1}) \cdots P(x_{n} | x_{1}, \dots , x_{n-1}).$$

Transformer 구조는 최근에 언어 모델링의 사실상 표준이 되었습니다. Transformer 기반의 언어 모델의 가장 중요한 요소는 **self-attention** 레이어입니다. input hidden state 시퀀스 \\( (x_{1},\dots,x_{n}) \in \mathbb{R}^{n \times d} \\) 에 대하여, self-attention 레이어는 우선 각 position \\(i\\) 에 대해 선형 변환을 적용하여 query, key, value 벡터를 뽑아냅니다.

$$q_{i}=W_{q}x_{i}, k_{i}=W_{k}x_{i}, v_{i}=W_{v}x_{i}.$$

그 후, self-attention 레이어는 한 위치에 있는 query 벡터를 모든 key 벡터와 곱함으로써 attention score인 \\(a_{ij}\\)를 계산합니다. 그 후 value 벡터를 이용해서 최종적으로 가중치 평균인 \\(o_{i}\\) 를 계산합니다.

$$a_{ij} = \frac{\exp\left(\frac{q_i^\top k_j}{\sqrt{d}}\right)}{\sum_{t=1}^i \exp\left(\frac{q_i^\top k_t}{\sqrt{d}}\right)}, \quad o_i = \sum_{j=1}^i a_{ij} v_j.$$

이 연산 이외에도 Transformer 모델을 이루는 레이어들에서도 각각 연산들이 이루어집니다.

### 2.2 LLM Service & Autoregressive Generation

LLM은 한번 훈련되면, 조건부 생성 서비스로 배포됩니다(e.g. 완성형 API, 챗봇). LLM 서비스에 들어가는 인풋은 input prompt 토큰들(\\(x_{1},\dots,x_{n}\\))로 이루어져 있고, LLM 서비스는 output 토큰들(\\(x_{n+1},\dots,x_{n+T}\\))을 생성합니다. 이것들을 시퀀스라고 합니다.

LLM은 오로지 새로운 토큰을 한 개씩 생성할 수 있고, 새로운 토큰은은 이전에 생성된 모든 토큰들에(특히 query, key 벡터들에) 의존합니다. 순차적으로 생성하는 과정에서, 존재하는 토큰들의 key 와 value 벡터들은 캐싱되는데, 이것을 **KV cache** 라고 합니다. 따라서 하나의 토큰의 KV cache 는 이전의 모든 토큰들에 의존합니다. 이것은 여러 위치에서 등장하는 같은 토큰의 KV cache 가 다름을 의미합니다.

요청 프롬프트가 주어질 때, LLM 서비스에서 토큰을 생성하는 과정은 2 단계로 나눌 수 있습니다.

 첫번째로, **prompt phase** 에서는 유저의 프롬프트 \\(x_{1},\dots,x_{n}\\) 를 가지고 새로운 토큰 \\(P(x_{n+1} | x_{1},\dots,x_{n})\\)  의 확률을 계산합니다. 이 과정 내에서는 key 벡터들 \\(k_{1},\dots,k_{n}\\) 과 value 벡터들 \\(v_{1},\dots,v_{n}\\) 도 생성합니다. 프롬프트 토큰 \\(x_{1},\dots,x_{n}\\) 는 모두 알고 있기 때문에, 이 과정은 행렬 연산 최적화로 병렬화할 수 있습니다. 따라서 GPU 를 활용하여 효율적으로 병렬 연산을 진행할 수 있습니다.

 두번째로, **autoregressive generation phase**는 남아있는 새로운 토큰들을 순차적으로 생성합니다. 이터레이션 \\(t\\) 에서, 모델은 하나의 토큰 \\(x_{n+t}\\) 를 인풋으로 받고, key 벡터들  \\(k_{1},\dots,k_{n+t}\\) 와 value 벡터들 \\(v_{1},\dots,v_{n+t}\\) 를 사용해서 확률 \\(P(x_{n+t+1} | x_{1},\dots,x_{n+t})\\) 를 계산합니다. 여기서 1에서 n+t-1 의 위치에 있는 key, value 벡터들은 이전의 이터레이션에서 캐싱되고, 새로운 key, value 벡터인 \\(k_{n+t}\\) 와 \\(v_{n+t}\\) 만 연산됩니다. 이 단계는 시퀀스가 최대 길이에 도달하거나, end-of-sequence(\<eos\>) 토큰을 뱉어낼때까지 계속됩니다. 각 이터레이션마다 연산이 병렬화될 수 없는데, 왜냐하면 데이터 의존성 문제와 병렬화에 덜 효율적인 매트릭스-벡터 연산을 사용하기 때문입니다. 따라서 이 단계는 GPU 를 상당히 비효율적으로 사용하며, memory-bound 가 되는데, 이것은 하나의 요청에서 latency 의 상당히 큰 비중을 차지하는 원인이 됩니다.










## Reference

- Kwon, Woosuk, Zhuohan Li, Siyuan Zhuang, Ying Sheng, Lianmin Zheng, Cody Hao Yu, Joseph E. Gonzalez, Hao Zhang, and Ion Stoica. “Efficient Memory Management for Large Language Model Serving with PagedAttention.” arXiv, September 12, 2023. https://doi.org/10.48550/arXiv.2309.06180.
