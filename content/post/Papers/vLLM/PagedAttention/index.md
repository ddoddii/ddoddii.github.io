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

### 2.3 Batching Techniques for LLMs

LLM을 서빙하는 것은 여러 개의 요청을 배칭함으로써 컴퓨팅 효율을 높일 수 있습니다. 왜냐하면 요청들은 같은 모델 가중치를 공유하므로, 매번 가중치들을 다시 로드할 필요가 없습니다. 여러 개의 요청을 한 번에 처리하는 배치(batch) 단위로 묶으면, 이 가중치 이동 비용이 여러 요청에 나뉘어 부담되므로(=amortized) 개별 요청 당 부담이 줄어들게 됩니다. 배치 크기가 충분히 크다면, 즉 한 번에 많은 요청을 처리한다면, 가중치 이동 오버헤드보다 연산 오버헤드가 더 지배적이 됩니다.
즉, 모델 가중치를 여러 요청이 공유하기 때문에 가중치 이동 비용이 상대적으로 작아지고, 대신 대량의 연산을 수행하는 비용이 더 커집니다. 

그렇지만 **LLM 서비스에서 배칭을 사용하는 것은 아래 2가지 이유로 인해 쉬운 문제가 아닙니다.**  첫번째로, **요청이 다른 시간대에 도착**할 수 있습니다. 단순한 배칭 전략은, 먼저 도착한 요청이 나중 요청을 기다리게 하거나, 새로운 요청이 이전 요청이 끝날 때까지 기다리게 만들어, 심각한 대기 지연(queuing delay)을 초래할 수 있습니다.  두 번째 이유는 **요청들의 입력과 출력 길이가 매우 다를 수 있기 때문**입니다. 단순한 배칭 기법은 요청들의 입력과 출력을 동일한 길이로 맞추기 위해 패딩(padding)을 추가해야 하므로, GPU 연산과 메모리를 낭비하게 됩니다. 즉, 효율적인 배칭 전략을 설계해야 하고, 단순한 방법으로는 성능이 좋지 않습니다.

이 문제를 해결하기 위해, 세밀한 배칭 기법(fine-grained batching mechanisms) 인 **셀룰러 배칭(cellular batching)** 과 **이터레이션 단위 스케줄링(iteration-level scheduling)** 같은 방법이 제안되었습니다. **기존의 요청 단위(request level) 방식과 달리, 이러한 기법들은 이터레이션 단위(iteration level) 에서 동작합니다.** 

- 기존 방법: 요청 단위로 배치를 구성하여, 하나의 배치가 끝날 때까지 새로운 요청을 기다려야 함.
- 새로운 방법: 개별 이터레이션(iteration, 반복 연산) 단위에서 요청을 조정하여, 배치가 끝날 때까지 기다릴 필요 없음.

각 이터레이션이 끝난 후, 완료된 요청은 배치에서 제거되고, 새로운 요청이 추가됩니다. 즉, 고정된 배치가 아니라, 매 이터레이션마다 배치가 동적으로 변합니다.
요청이 끝날 때까지 기다릴 필요 없이, 바로 다음 요청을 추가할 수 있습니다. 따라서, 새로운 요청은 전체 배치가 끝날 때까지 기다리지 않고, 단 한 번의 이터레이션 후에 처리될 수 있습니다. 즉, 대기 시간이 줄어듭니다.

```less
요청 3개 (A, B, C)
A: 2번 연산 필요
B: 4번 연산 필요
C: 3번 연산 필요
새로운 요청 D(3번 연산 필요)가 Iteration 2에서 도착

Time Step | A  | B  | C  | D  
----------|----|----|----|---
Iteration 1 | ✅ | ✅ | ✅ |    
Iteration 2 | ✅ | ✅ | ✅ | ✅  
Iteration 3 |    | ✅ | ✅ | ✅  
Iteration 4 |    | ✅ |    | ✅  

Iteration-Level Batching 동작 흐름:

A, B, C를 묶어서 배치 실행
A가 끝나면 즉시 새로운 요청 D를 추가
배치 내에서 연산이 끝난 요청은 제거하고, 새로운 요청을 계속 추가함.
```

또한, **특수한 GPU 커널을 사용하면 입력과 출력의 패딩이 필요하지 않습니다.** 큐 대기 지연(queuing delay)과 패딩으로 인한 비효율을 줄임으로써, 세밀한 배칭 기법(fine-grained batching) 은 LLM 서비스의 처리량(throughput)을 크게 향상시킵니다. 

## 3. Memory Challenges in LLM Serving

세밀한 배칭(fine-grained batching)은 연산 낭비를 줄이고 요청을 더 유연하게 배치할 수 있게 하지만, 동시에 처리할 수 있는 요청 수는 여전히 GPU 메모리 용량에 의해 제한된다. 특히, KV 캐시(KV cache) 저장 공간이 주요 제한 요소가 됩니다. 즉, 서빙 시스템의 병목은 연산 능력(CUDA 코어 성능)이 아니라, 메모리 사용량이 됩니다(memory-bound). 이러한 메모리 한계를 극복하려면, 다음과 같은 메모리 관리의 3가지 도전 과제를 해결해야 합니다.

<span style="font-size:120%">1. **Large KV cache**</span>

KV 캐시 사이즈는 요청 수가 많아짐에 따라 급격히 늘어납니다. 요청이 많아질수록, 또는 개별 요청의 토큰 수가 늘어날수록 KV 캐시가 GPU 메모리를 엄청나게 차지하게 됩니다. 13B(130억) 파라미터를 가진 OPT 모델에서 토큰당 KV 캐시 크기를 계산해보면, 2(key & value 벡터의 크기) * 5120(hidden state size) * 40(num of layers) * 2(bytes per FP16) ≈ 800KB 입니다. OPT가 최대 2048개의 토큰을 생성할 수 있으므로, 요청 하나 당 KV 캐시 크기는 최대 1.6GB에 달할 수 도 있습니다. 현재 사용 가능한 GPU의 메모리는 수십 GB 수준이기 때문에, 모든 메모리를 KV 캐시에 할당하더라도, 수십 개 요청밖에 처리할 수 없습니다. 

게다가, 비효율적인 메모리 관리로 인해 배치 크기가 더욱 감소할 수 있습니다. GPU 메모리가 단순히 부족한 것뿐만 아니라, 비효율적인 메모리 할당(e.g. 조각화, 불필요한 데이터 유지 등) 도 배치 크기를 제한하는 원인이 됩니다.

추가적으로, 현재 추세를 보면, GPU의 연산 속도는 메모리 용량보다 더 빠르게 증가하고 있습니다. 예를 들어, NVIDIA A100에서 H100으로 넘어가면서 FLOPS는 2배 이상 증가했지만, GPU 메모리는 여전히 최대 80GB 수준에 머물러 있습니다. 현재 추세대로라면, LLM 서빙의 주요 병목은 GPU 연산 능력이 아니라 "**메모리 관리**"가 될 것입니다.


<span style="font-size:120%">2. **Complex decoding algorithms**</span>

LLM 서비스는 사용자에게 다양한 디코딩 알고리즘(decoding algorithms) 을 제공하며, 각 알고리즘은 메모리 관리의 복잡성에 서로 다른 영향을 미칩니다. 디코딩 알고리즘에는 Greedy decoding (탐욕적 디코딩), Top-k sampling, Top-p (nucleus) sampling, Beam search (빔 서치), Temperature scaling 등이 있는데, 각 디코딩 방식에 따라 KV 캐시 사용량과 공유방식이 달라집니다. 예를 들어, **사용자가 단일 입력 프롬프트에서 여러 개의 랜덤 샘플을 요청하는 경우, 프롬프트 부분의 KV 캐시를 공유할 수 있습니다.** 이는 실험에서 전체 KV 캐시 메모리의 12% 를 차지하며, 이를 공유하면 메모리 사용량을 줄일 수 있습니다. 

반면, **자동 회귀 생성(autoregressive generation) 단계의 KV 캐시는 공유할 수 없습니다**. 각 샘플 결과가 서로 다르고, 각 샘플의 KV 캐시가 서로 다른 문맥(context)과 위치(position)에 의존하기 때문입니다.

💡 <u>왜 Autoregressive Generation에서는 KV 캐시를 공유할 수 없을까?</u>

- Autoregressive Generation은 이전 토큰을 기반으로 다음 토큰을 예측하는 방식임.
- 따라서, 샘플링을 여러 번 수행하면 각 샘플의 KV 캐시가 서로 다르게 변형됨.
- 즉, 각 샘플의 결과가 달라질 수 있기 때문에 KV 캐시를 공유할 수 없음.
- 결과적으로 프롬프트 부분(초기 입력)은 공유할 수 있지만, 이후 생성된 토큰들의 KV 캐시는 개별적으로 저장해야 함.

**KV 캐시 공유의 범위는 사용된 디코딩 알고리즘의 종류에 따라 다릅니다.** 예를 들어, Beam Search 같은 정교한 디코딩 알고리즘에서는, 서로 다른 빔(beam)들이 더 큰 부분의 KV 캐시를 공유할 수 있습니다. 실험 결과, 최대 55%의 메모리 절약 효과가 있었으며, 공유 방식은 디코딩 과정이 진행됨에 따라 동적으로 변화합니다.

💡 <u>Beam Search에서 KV 캐시 공유가 가능한 이유?</u>  

- Beam Search 는 하나의 입력 프롬프트에서 여러 후보 문장을 동시에 탐색하는 기법.
- 즉, 초기 몇 단계까지는 서로 유사한 후보 문장을 유지하므로, 일정 부분 KV 캐시를 공유 가능.
- 실험 결과, Beam Search의 경우 최대 55%까지 KV 캐시를 공유할 수 있어 메모리 절약 가능.
- 하지만 디코딩이 진행되면서(문장이 길어질수록) 각 빔이 서로 다른 문맥으로 갈라지므로, 공유 비율이 점점 줄어듦.
 

<span style="font-size:120%">3. **Scheduling for unknown input & output lengths**</span>  

LLM 서비스의 요청들은 입력과 출력 길이에 있어서 가변성을 가집니다. 

💡 <u>문제점</u>:

- 입력/출력 길이가 예측 불가능하기 때문에, 고정된 방식으로 메모리를 할당하면 비효율적임.
- 따라서 유연한 메모리 관리 및 스케줄링이 필요함.

따라서, 메모리 관리 시스템은 다양한 입력 길이를 수용할 수 있어야 합니다. 또한, 디코딩이 진행되면서 출력 길이가 늘어날수록 KV 캐시의 메모리 사용량도 증가하며, 이는 새로운 요청을 처리할 메모리를 소진시키거나 기존 요청의 생성 과정에서 메모리 부족을 유발할 수 있습니다. 

따라서 시스템은 일부 요청의 KV 캐시를 GPU 메모리에서 삭제하거나 스왑하는 등의 스케줄링 결정을 내려야 합니다. 

💡 <u>해결책</u>:

- KV 캐시 삭제(Discarding KV Cache)
  - 특정 요청의 응답이 끝나면 즉시 KV 캐시를 삭제하여 메모리를 확보.
  - 단점: 삭제된 KV 캐시는 다시 사용할 수 없음 → 이전 문맥 유지 불가능.
- KV 캐시 스왑(Offloading KV Cache to CPU or NVMe SSD)
  - GPU 메모리가 부족하면 일부 KV 캐시를 CPU 메모리(RAM)나 SSD로 이동.
  - 필요할 때 다시 불러오면 연산을 지속할 수 있음.
  - 단점: 스왑 속도가 GPU 메모리보다 느려서 성능 저하 발생 가능.
- 출력 길이 예측 & 동적 할당 (Adaptive Allocation)
  - 과거 데이터를 분석하여, 요청별 평균적인 출력 길이를 예측하고 메모리를 동적으로 할당하는 기법.
  - 너무 짧게 할당하면 추가 메모리 필요 시 스왑 필요, 너무 길게 할당하면 메모리 낭비 발생.

### 3.1 Memory Management in Existing Systems

현재 딥러닝 프레임워크의 대부분의 연산자(operator)는 텐서를 연속적인(contiguous) 메모리 블록에 저장해야 합니다. 따라서, 기존 LLM 서빙 시스템에서도 하나의 요청의 KV 캐시를 연속적인 텐서 형태로 저장합니다. 

💡 <u>연속적인 메모리 할당(Contiguous Allocation)이란?</u>

- GPU 메모리는 연속적인 블록으로 할당하는 것이 연산 속도를 높이는 데 유리함.
- 따라서, 기존 LLM 서빙 시스템에서는 각 요청(Request)의 KV 캐시를 하나의 커다란 연속적인 텐서(Tensor)로 저장함.
- ✅ 장점: 연산 최적화 (메모리 접근 속도 빠름).
- 🚨 단점: 가변적인 출력 길이를 다루기에 비효율적.

LLM의 출력 길이를 미리 예측할 수 없기 때문에, 기존 시스템에서는 각 요청의 최대 가능 시퀀스 길이(Maximum Possible Sequence Length) 를 기준으로 메모리를 정적으로 할당합니다. 즉, 실제 입력 길이나 최종 출력 길이와 관계없이 미리 큰 메모리 블록을 할당합니다.

💡 <u>문제점</u>:

- 요청마다 출력 길이가 다르지만, 메모리를 최악의 경우(최대 길이)에 맞춰 미리 할당함.
- ex)
  - 요청 A: 최대 2048개 토큰을 출력할 수 있음 → 2048개 토큰만큼 메모리 할당
  - 요청 B: 최대 512개 토큰을 출력할 수 있음 → 512개 토큰만큼 메모리 할당
  - 실제 사용량이 적더라도 최대 할당량만큼 메모리가 낭비됨.
- ✅ 장점: 메모리 관리가 단순함.
- 🚨 단점: 불필요한 메모리 낭비 발생

기존 시스템의 정적 할당 방식은 세 가지 주요한 메모리 낭비의 원인을 초래합니다. 

<span style="font-size:120%">1. **예약된 슬롯 (Reserved Slots for Future Tokens)**</span> 

- LLM이 생성할 최대 토큰 수만큼 미리 메모리를 예약하지만, 모든 요청이 최대 길이까지 출력을 생성하지 않으므로 메모리가 낭비됩니다.


<span style="font-size:120%">2. **내부 단편화 (Internal Fragmentation)**</span>  

- 요청별 최대 시퀀스 길이에 맞춰 메모리를 과잉 할당(over-provisioning)하여 발생합니다. 
  - 요청이 할당된 최대 시퀀스 길이를 다 쓰지 않으면,
  - 사용되지 않은 메모리가 내부적으로 남아 있음 → 결국 메모리 낭비.

<span style="font-size:120%">3. **외부 단편화 (External Fragmentation)**</span>  

- 버디 할당기(Buddy Allocator) 같은 메모리 할당 방식에서 외부 단편화가 발생합니다.
  - 메모리 할당기는 여러 요청의 크기에 맞춰 메모리를 나누어 할당하는데,
  - 요청들이 끝나고 메모리가 해제될 때, 남는 작은 블록들이 생기면서 메모리 사용이 비효율적이 됨.


외부 단편화는 생성된 토큰을 위해 절대 사용되지 않으며, 이는 요청을 처리하기 전부터 예측 가능합니다. 내부 단편화도 마찬가지로 사용되지 않으며, 이는 요청이 샘플링을 마친 후에야 확인 가능하다. 두 가지 모두 순수한 메모리 낭비입니다. 저자들의 실험 결과, 기존 시스템에서 실제로 효과적으로 사용되는 GPU 메모리는 20.4% 수준까지 낮아질 수 있다고 합니다. 

**Compaction 기법**이 단편화(Fragmentation) 문제를 해결할 수 있는 잠재적인 방법으로 제안되기도 했습니다.

💡 <u>Compaction이란?</u>

- 메모리 조각화(Fragmentation)를 줄이기 위해, 분산된 데이터를 연속된 공간으로 다시 정렬하는 기법.
- 쉽게 말해 메모리를 "정리"하여 비효율적인 빈 공간을 제거하는 것.
- 기존 시스템에서는 Garbage Collection(GC) 과정에서 메모리 Compaction을 활용하기도 함.

- ✅ Compaction의 기대 효과
  - 내부 & 외부 단편화 문제 해결 → 작은 조각난 메모리를 합쳐서 활용 가능.
  - 메모리 사용 효율 증가 → 낭비되는 공간을 최소화하여 더 많은 요청을 처리 가능.

그러나 LLM 서빙 시스템은 성능이 중요한 환경이므로, Compaction을 적용하는 것이 비현실적입니다. 특히, KV 캐시의 크기가 너무 커서 Compaction이 어렵습니다.

💡 <u>왜 Compaction이 LLM 서빙에서는 비효율적일까?</u>

- KV 캐시는 너무 커서 이동 비용이 큼
  - Compaction을 수행하려면 KV 캐시 데이터를 새로운 연속된 공간으로 이동해야 함.
  - 하지만 KV 캐시는 한 요청당 최대 1.6GB에 달할 정도로 크기 때문에,
  - 여러 요청의 KV 캐시를 이동하는 것은 막대한 연산 비용을 초래함.
  - 실시간 응답이 중요한 LLM 서빙에서는 속도가 중요

- LLM 서빙 시스템은 사용자 요청을 빠르게 처리해야 하는 실시간 서비스임.
  - 하지만 Compaction을 수행하는 동안 기존 요청을 멈추고 데이터를 재배치해야 하므로 응답 속도가 저하됨.
  - 즉, 추가적인 연산이 성능 저하를 유발하기 때문에 적용이 어려움.

Compaction을 적용한다고 해도, 기존의 정적 메모리 할당 방식은 여전히 문제이며, 디코딩 알고리즘에 맞춘 메모리 공유를 방해합니다. 


## 4. Method

이 연구에서는 새로운 어텐션 알고리즘, **PagedAttention**을 개발하고, 이를 기반으로 한 LLM 서빙 엔진 **vLLM** 을 설계하여, #3 에서 제시된 문제들을 해결합니다. vLLM의 아키텍처는 Fig. 4에 나타나 있으며, vLLM은 중앙 스케줄러(Centralized Scheduler)를 사용하여 여러 GPU 워커(GPU Workers)의 실행을 조정합니다. 

<img width="405" alt="Image" src="https://github.com/user-attachments/assets/ea90f5ce-804e-494e-b5a3-ec88e0534acc" />

💡 <u>vLLM 아키텍처 개요</u>

- 중앙 스케줄러(Centralized Scheduler)
  - 여러 GPU에서 분산 실행(distributed execution)을 효율적으로 조정하는 역할.
  - KV 캐시 관리와 요청 스케줄링을 담당함.
  - 기존 방식보다 더 유연한 배칭 및 요청 처리가 가능하도록 설계됨.
- 분산 GPU 워커(Distributed GPU Workers)
  - 모델 추론을 수행하는 GPU 작업자들.
  - PagedAttention을 활용하여 메모리를 더 효율적으로 사용함

KV 캐시 관리자는 PagedAttention을 통해 KV 캐시를 페이지 기반(Paged) 방식으로 효과적으로 관리합니다. 세부적으로, 중앙 스케줄러가 보낸 명령을 기반으로, GPU 워커에서 KV 캐시 메모리를 물리적으로 관리합니다. 

### 4.1 PagedAttention

PagedAttention은 운영 체제의 **페이징(Paging)** 개념에서 영감을 받아 설계된 어텐션 알고리즘이며, §3에서 설명한 메모리 문제를 해결합니다.

💡 <u>페이징(Paging)이란?</u>

- 운영체제(OS)에서 메모리를 작은 "페이지(Page)" 단위로 나누어 관리하는 방식.
- 연속적인 메모리 할당이 필요하지 않아, 단편화 문제를 줄이고 메모리를 효율적으로 활용할 수 있음.
- PagedAttention은 이 개념을 LLM의 KV 캐시 관리에 적용한 것.

- ✅ 기존 어텐션 방식의 문제점

  - 기존 어텐션 알고리즘은 KV 캐시를 연속적인(Contiguous) 메모리 공간에 저장해야 함.
  - 하지만, 입출력 길이가 다르면 KV 캐시 단편화(Fragmentation)가 발생하여 메모리 낭비가 큼.

- ✅ PagedAttention이 해결하는 방법
  - KV 캐시를 작은 블록(KV Blocks)으로 나누어 저장하고, 필요할 때만 불러옴.
  - 이를 통해 KV 캐시가 반드시 연속적인 메모리 공간에 있을 필요가 없어지고, 더 유연한 메모리 관리 가능.

기존 어텐션 알고리즘과 달리, PagedAttention은 연속적인 키/값 벡터를 "**비연속적인(non-contiguous)**" 메모리 공간에 저장할 수 있습니다. 세부적으로, PagedAttention은 KV 캐시를 "**KV 블록(KV Blocks)**"으로 분할하며, 각 블록은 일정 개수(B)의 토큰에 대한 키(Key)와 값(Value) 벡터를 포함합니다. 이때 B를 **KV block size** 라고 합니다. 

- key block \\( K_j = \left( k_{(j-1)B+1}, \dots, k_{jB} \right) \\)
- value block \\( V_j = \left( v_{(j-1)B+1}, \dots, v_{jB} \right) \\)

기존 어텐션 연산을 KV 블록 단위로 수행하여, 필요할 때만 메모리에 접근합니다. 

💡 <u>기존 어텐션(Scaled Dot-Product Attention) 과 비교하면?</u>

1. **Scaled Dot-Product Attention**

기본적인 Scaled Dot-Product Attention 의 공식은 아래와 같다.

$$
A_i = \text{softmax} \left( \frac{Q_i K^T}{\sqrt{d}} \right)
$$

$$
o_i = V^T A_i
$$

여기서,

- \\( Q_i \\) : 쿼리 벡터 (Query Vector)
- \\( K \\) : 키 벡터 (Key Vector)
- \\( V \\) : 값 벡터 (Value Vector)
- \\( d \\) : 히든 차원 (Hidden Dimension)

- ✅ 기존 방식의 문제점:
  - 전체 K와 V를 한꺼번에 연산해야 함 → 큰 메모리 필요.
  - 메모리가 부족하면 한 번에 처리할 수 있는 요청 수가 제한됨.

2. **PagedAttention에서 블록 단위 연산으로 변환**

(1) 블록 단위 Attention Score 계산

$$
A_{ij} =
\frac{\exp\left( q_i^T K_j / \sqrt{d} \right)}
{\sum_{t=1}^{\lceil i / B \rceil} \exp\left( q_i^T K_t / \sqrt{d} \right)}
$$

- 각 기호의 의미
  - \\( A_{ij} \\) : 쿼리 \\( q_i \\) 가 KV 블록 \\( j \\) 에 대해 가지는 어텐션 점수 (Attention Score)
  - \\( K_j \\) : \\( j \\)번째 KV 블록의 Key 벡터들
  - \\( d \\) : Hidden 차원 수 (차원을 정규화하기 위해 제공된 사용)
  - \\( B \\) : KV 블록 크기 (한 블록에 포함되는 토큰 개수)
  - \\( \lceil i / B \rceil \\) : 쿼리 \\( q_i \\) 가 포함된 블록까지의 총 블록 개수

- 기존 어텐션에서는 전체 Key \\( K \\) 를 한꺼번에 계산했지만,  
- PagedAttention에서는 각 블록 \\( K_j \\) 별로 나누어 연산함.  
- Softmax 계산의 분모도 이전 블록들까지의 Key 벡터만 포함하여 부분적으로 수행됨.  

(2) 블록 단위로 Value 벡터를 활용하여 최종 출력 계산

$$
o_i = \sum_{j=1}^{\lceil i / B \rceil} V_j A_{ij}^T
$$

- 각 기호의 의미
  - \\( o_i \\) : 쿼리 \\( q_i \\) 에 대한 최종 어텐션 출력 (Output Vector)
  - \\( V_j \\) : \\( j \\)번째 KV 블록의 Value 벡터들
  - \\( A_{ij}^T \\) : 쿼리 \\( q_i \\) 와 KV 블록 \\( j \\) 간의 어텐션 점수 (Transpose 연산 포함)

- 어텐션 점수 \\( A_{ij} \\) 가 계산되면, 이를 활용하여 Value 벡터 \\( V_j \\) 와 곱하여 최종 출력을 생성.
- 기존 방식에서는 전체 \\( V \\) 를 한꺼번에 곱했지만,  
- PagedAttention에서는 블록 단위로 Value 벡터를 곱하여 최종 출력을 생성함.


<img width="400" alt="Image" src="https://github.com/user-attachments/assets/52d93b97-90c6-4c54-8b9c-97dee0843421" />

어텐션 연산 중, PagedAttention 커널은 필요한 KV 블록을 식별하고, 개별적으로 불러옵니다. Fig. 5 의 예시에서 볼 수 있듯이, Key/Value 벡터가 3개의 블록으로 분할되어 있고, 이 블록들은 물리적 메모리에서 연속적이지 않습니다. 커널은 쿼리 토큰(forth)의 query 벡터 \\(q_{i}\\) 를 각 블록의 key 벡터와 곱하여 어텐션 점수 \\(A_{ij}\\) 를 계산합니다. 

- Block 0 : "Four", "score", "and", "seven" -> \\(K_{0}\\)
- Block 1 : "years", "ago", "our", "fathers" -> \\(K_{1}\\)
- Block 2 : "brought", "forth" -> \\(K_{2}\\)

그 후에 \\(A_{ij}\\) 를 value 벡터 \\(V_{j}\\) 와 곱하여 마지막 결과인 \\(o_{i}\\) 를 연산합니다.

결론적으로, **PagedAttention은 KV 블록을 비연속적인 물리적 메모리에 저장할 수 있도록 하며, 이를 통해 vLLM에서 더 유연한 페이지 기반(Paged) 메모리 관리를 가능하게 합니다.**

### 4.2 KV Cache Manager

vLLM의 메모리 관리 핵심 개념은 운영 체제(OS)의 가상 메모리(Virtual Memory) 개념과 유사합니다.

💡 <u>가상 메모리(Virtual Memory)란?</u>

- 운영 체제(OS) 는 메모리를 고정 크기의 "페이지(Page)" 단위로 관리.
- 프로세스의 논리적 주소(Logical Address)를 물리적 주소(Physical Address)로 매핑하여,
- 연속적인 논리 메모리를 제공하면서도 실제 물리 메모리는 분산 저장 가능.
- 또한, 필요할 때만 물리적 메모리를 할당하여 미리 예약할 필요가 없음.

✅ <u>vLLM이 이 개념을 KV 캐시 관리에 적용하는 방법</u>:

- KV 캐시를 고정 크기의 "KV 블록(KV Blocks)"으로 분할하여 관리.
- 논리적인 KV 블록(Logical KV Blocks)과 물리적인 KV 블록(Physical KV Blocks)을 분리하여 저장.
- 연속적인 논리 KV 블록이 실제로는 비연속적인 물리 메모리에 저장될 수 있도록 설계.
- 이를 통해 메모리를 미리 예약할 필요 없이, 동적으로 확장 가능!

PagedAttention을 활용하여, vLLM은 KV 캐시를 가상 메모리의 페이지와 유사한 고정 크기 **KV 블록(KV Blocks)** 으로 구성합니다. 

💡 <u>KV 캐시 관리 방식</u>

1. 논리 KV 블록(Logical KV Blocks)

   - 각 요청(Request)의 KV 캐시는 논리적인 KV 블록들로 표현됨.
   - 새 토큰이 생성될 때마다 왼쪽에서 오른쪽으로 채워짐.
   - 마지막 블록의 남은 공간은 미래의 토큰 생성을 위해 예약됨.

2. 물리 KV 블록(Physical KV Blocks)

   - GPU 워커(GPU Workers) 에서 블럭 엔진(block engine)이 KV 블록을 실제 물리 메모리(GPU DRAM)에 할당.
   - 필요할 때만 할당하여, 미리 예약할 필요가 없음 → 메모리 낭비 최소화!
   - 일부 KV 블록은 CPU RAM에 저장되었다가 필요할 때 GPU로 스왑(Swapping) 가능 (§4.5에서 설명).

3. KV 블록 테이블(Block Table) 유지

   - 논리 KV 블록과 물리 KV 블록 간의 매핑 정보를 저장.
   - 각 요청에 대해 어떤 논리 블록이 어떤 물리 블록에 매핑되는지 기록.
   - 블록별 사용량(빈 공간 포함)도 함께 관리하여 동적 확장 가능.


논리 KV 블록과 물리 KV 블록을 분리하면, 모든 위치를 미리 예약할 필요 없이 KV 캐시 메모리를 동적으로 확장할 수 있습니다. 이를 통해 기존 시스템의 메모리 낭비를 대부분 제거할 수 있습니다.

### 4.3 Decoding with PagedAttention and vLLM

vLLM은 기존처럼 최대 시퀀스 길이만큼 미리 메모리를 예약할 필요가 없습니다. vLLM에서는 최대 길이만큼 메모리를 미리 할당하지 않고, 필요한 만큼만 KV 블록을 할당합니다. 

<img width="668" alt="Image" src="https://github.com/user-attachments/assets/6e16fe62-6bbe-4e5e-910e-05b8288a4b6f" />

💡 <u>KV 블록 테이블과 논리-물리 블록 매핑</u>

위의 Fig. 6 예시를 보면서 디코딩 과정 중에서 어떻게 KV 캐시를 관리하는지 봅시다.

1. 초기 프롬프트 처리 단계

   - 프롬프트 입력: "Four score and seven years ago our"
   - 프롬프트 처리 방식:
     - 첫 번째 논리 KV 블록(0): "Four score and seven" → 물리 블록 7에 저장
     - 두 번째 논리 KV 블록(1): "years ago our fathers" → 물리 블록 1에 저장
     - prefill 단계에서는 vLLM 은 프롬프트와 첫번째 아웃풋 토큰의 KV 캐시는 기존의 self-attention 알고리즘으로 계산함. 
     - vLLM은 첫번째 4개 토큰의 KV 캐시를 논리 블록 0 에 저장하고, 나머지 3개 토큰의 KV 캐시를 논리 블록 1에 저장함.
     - 남은 공간은 이후의 생성 과정(Autoregressive Decoding)을 위해 예약됨

2. 첫번째 디코딩 단계 : 새 토큰 생성

   - 첫 번째 디코딩 단계에서, PagedAttention 알고리즘이 물리 블록 7과 1에서 연산을 수행하여 새로운 토큰을 생성합니다. 
     - 새로운 토큰: "fathers"
     - 기존 KV 블록 1에 빈 공간이 있어서 새로운 토큰의 KV캐시를 그 안에 저장.
     - 블록 테이블이 업데이트 됨.

3. 두 번째 디코딩 단계: 새로운 KV 블록 할당

   - 두 번째 디코딩 단계에서, 마지막 논리 블록이 가득 차면 새로운 논리 블록을 할당하고, 이에 대응하는 새로운 물리 블록을 배정합니다.
     - 새로운 토큰: "brought"
     - 논리 블록2가 새롭게 추가되고, 이에 대응하는 물리 블록3이 할당됨.
     - Block Table이 업데이트됨 → 논리 블록 2가 물리 블록 3과 매핑됨

하나의 KV 블록에 여러 토큰을 저장하면, PagedAttention 커널이 여러 위치의 KV 캐시를 동시에 처리할 수 있어 하드웨어 활용도를 높이고 지연 시간을 줄일 수 있습니다.

✅ <u>KV 블록 크기를 조정하여 성능 최적화 가능</u>

- 작은 블록 크기 → 메모리 활용도 증가하지만, 연산 효율이 낮음.
- 큰 블록 크기 → 연산 효율이 높지만, 메모리 단편화가 발생할 수 있음.
- vLLM은 블록 크기를 조정하여 하드웨어 성능과 메모리 활용 간의 균형을 맞춤.

이전의 블록들이 모두 찼을때만 물리 블록을 더 할당함으로써 vLLM은 메모리 낭비를 방지할 수 있습니다. 이것은 배칭을 위해서 더 많은 요청을 끼워넣을 수 있다는 것을 의미하고, 처리량(throughput)의 증가로 이어집니다. 요청이 생성을 끝내면, KV 블럭은 메모리 할당 해제되고, 다른 요청의 KV 캐시를 저장할 수 있습니다.

<img width="576" alt="Image" src="https://github.com/user-attachments/assets/a6b4770b-7969-4525-98e8-2b88d72bc1f7" />

Fig. 7에서는 두개의 요청 - Request A와 Request B의 KV 캐시가 어떻게 효율적으로 배치되는지를 보여줍니다. 두 시퀀스의 논리 블럭은 GPU worker 의 block engine 에 의해 각각 다른 물리 블럭으로 매핑됩니다. 이 논리 블럭들은 물리 메모리 내에서 연속적으로 할당될 필요가 없습니다. 따라서 GPU 메모리 내의 메모리와 공간은 효율적으로 활용될 수 있습니다.

### 4.4 Application to Other Decoding Scenarios

§4.3 에서 PagedAttention 과 vLLM 이 greedy 와 sampling 과 같이 하나의 유저 프롬프트를 인풋으로 받고, 하나의 출력 시퀀스를 만들어내는 기본적인 디코딩 알고리즘을 어떻게 다루는지 봤습니다. 그러나 LLM 어플리케이션은 더욱 복잡한 디코딩 시나리오도 지원을 하고 메모리 공유 기회도 지원해야 합니다. 이 섹션에서는 vLLM 이 어떻게 이것들을 지원하는지 살펴봅시다.

<span style="font-size:130%">1. **Parallel sampling**</span>

LLM 기반 프로그래밍 어시스턴트에서는 하나의 프롬프트에 대해 여러 개의 샘플링된 출력을 생성하고, 사용자가 원하는 출력을 선택할 수 있습니다. 

💡 <u>Parallel Sampling이란?</u>

- 하나의 프롬프트에서 여러 개의 샘플링된 출력을 동시에 생성하는 방식.
- 예제:
  - "Write a Python function for quicksort."
  - LLM이 3가지 다른 코드 버전을 생성하여 사용자에게 제공.

💡 <u>Parallel Sampling의 특징</u>

- 모든 샘플은 동일한 입력 프롬프트를 공유하지만, 이후 생성되는 내용은 서로 다를 수 있음.
- 즉, 초기 프롬프트의 KV 캐시는 여러 개의 샘플이 공유 가능함 → 메모리 최적화 기회가 많음.

지금까지는 하나의 요청이 단일 시퀀스를 생성하는 경우만 고려했습다. 이제부터는 하나의 요청이 여러 개의 시퀀스를 생성하는 일반적인 경우를 다룹니다. 

Parallel Sampling에서는 하나의 요청이 여러 개의 샘플을 포함하고 있으며, 이 샘플들은 동일한 입력 프롬프트를 공유합니다. 따라서 KV 캐시의 프롬프트 부분을 공유할 수 있습니다. PagedAttention과 페이지 기반 메모리 관리 기법을 활용하면 이러한 공유를 쉽게 실현할 수 있으며, 메모리를 절약할 수 있습니다.

<img width="667" alt="Image" src="https://github.com/user-attachments/assets/61892a37-899b-4db4-987d-9f561b9b0847" />

Fig. 8 에서는 2개 아웃풋의 parallel decoding 을 보여줍니다. 두 개의 출력이 동일한 프롬프트를 공유하므로, 프롬프트 단계에서 KV 캐시를 한 번만 저장하면 됩니다. 두 개의 샘플이 동일한 프롬프트를 사용하므로 논리 블록은 같은 물리 블록들에 매핑됩니다(논리 블록 0과 1은 물리 블록 7과 1에 각각 매핑됨). 하나의 물리 블록이 여러 개의 논리 블록에 매핑될 수 있으므로, 물리 블록마다 **참조 카운트(Reference Count)** 를 사용합니다. 이 경우에는 물리 블록 7과 1의 reference count 수는 모두 2입니다. 생성(generation) 단계에서는, 2개의 아웃풋은 다른 아웃풋 토큰을 출력하므로 서로 다른 KV 캐시 저장소가 필요합니다. 이때 vLLM 은 운영체제의 가상 메모리에서 사용하는 기법인 **Copy-on-Write(CoW)** 를 활용합니다. 

💡 <u>Copy-on-Write 란?</u>

- Copy-on-Write(CoW) 는 운영체제의 가상 메모리에서 사용하는 기법으로, 데이터가 변경되기 전까지 동일한 물리적 메모리를 공유하고, 변경이 발생하면 새로운 메모리를 할당하여 복사하는 방식.
- vLLM은 이 방식을 KV 캐시 관리에 적용하여, 불필요한 중복 저장을 방지함.


✅ <u>Fig. 8에서 Copy-on-Write의 동작 과정</u>

1. 초기 상태:

   - 샘플 A1과 A2는 동일한 물리 블록(7과 1)을 공유.
   - 참조 카운트(Reference Count) = 2 (두 개의 논리 블록이 공유 중).

2. 샘플 A1이 새로운 KV 캐시를 저장하려고 함

   - `"fathers"`를 저장해야 하는데, 현재 논리 블록 1이 물리 블록 1과 연결되어 있음.
   - 하지만, 이 블록을 A2도 공유 중이므로 직접 변경할 수 없음.

3. Copy-on-Write 발생 (복사 후 수정)
   - vLLM은 새로운 물리 블록(Physical Block 3)을 생성.
   - 기존 물리 블록(Physical Block 1)의 내용을 복사하여 새 블록에 저장.
   - 샘플 A1의 논리 블록 1을 새 물리 블록(Physical Block 3)과 연결.
   - 참조 카운트 감소: Physical Block 1의 참조 카운트 2 → 1.

4. 샘플 A2는 기존 블록을 그대로 사용
   - 샘플 A2가 "mothers"를 추가하려고 할 때, Physical Block 1의 참조 카운트가 이미 1이므로 직접 수정 가능.
   - 즉, Copy-on-Write(CoW)를 활용하면, 변경이 필요할 때만 새 블록을 할당하여 메모리를 최적화할 수 있음! 


결론적으로, 여러 샘플이 물리 블록을 공유함으로써 메모리 사용량이 크게 줄어들며, 특히 긴 입력 프롬프트의 경우 효과가 더욱 커집니다.

✅ <u>기존 방식 vs. vLLM 비교</u>

|기존 방식(메모리 낭비)|vLLM(PagedAttention + CoW)|
|----------------------|--------------------------|
|각 샘플이 별도로 KV 캐시를 저장해야 함 → 중복 데이터 증가|공유 가능한 KV 캐시는 하나만 저장 → 메모리 절약|
|변경되지 않은 데이터도 중복 저장됨|필요한 경우에만 새로운 블록을 생성|
|메모리 사용량이 크고, 처리 가능한 배치 크기 제한됨	|동일한 메모리에서 더 많은 요청을 처리 가능 (Throughput 증가)|




<span style="font-size:130%">2. **Beam Search**</span>

기계 번역과 같은 LLM 작업에서는 사용자가 가장 적절한 상위 k개의 번역 출력을 기대합니다. Beam Search는 Greedy Decoding(탐욕적 디코딩)이 단순히 매 단계에서 확률이 가장 높은 토큰을 선택하는 것과 달리, 여러 개의 후보군(beam)을 유지하면서 탐색을 진행하는 방식입니다. Beam Search는 현재 유지하는 후보군(beam)마다 가능한 모든 토큰을 고려하여 확률을 계산한 후, 가장 확률이 높은 k개의 후보를 유지합니다.


💡 <u>Beam Search란?</u>

1. 탐색 공간을 효율적으로 줄이면서도 더 나은 결과를 얻을 수 있음.
2. Beam Width \\(k\\): 한 번에 유지하는 후보 시퀀스의 개수를 결정하는 하이퍼파라미터.
   - \\(k=1\\)이면 Greedy Decoding과 동일함.
   - \\(k\\)가 클수록 더 다양한 결과를 탐색하지만, 연산량도 증가함.
3. 기본 동작 방식
   - 각 단계에서 모든 가능한 다음 토큰 후보를 고려.
   - 확률을 계산한 후, 가장 가능성이 높은 상위 k개 후보를 유지.
   - \\(k \cdot \vert V \vert \\) 개의 후보 중 상위 \\(k\\) 개를 선택하여 다음 단계로 진행 (\\(\vert V \vert\\)는 단어장 크기).




Parallel decoding과 달리, Beam Search는 초기 프롬프트 블록뿐만 아니라, 디코딩 과정에서 동적으로 변화하는 다양한 블록을 공유할 수 있습니다.

✅ <u>Beam Search에서 KV 캐시 공유 방식이 중요한 이유</u>

- Parallel Sampling에서는 입력 프롬프트의 KV 캐시만 공유하고, 이후 각 샘플이 개별적으로 디코딩됨.
- Beam Search에서는 초기 프롬프트뿐만 아니라, 중간 블록들도 여러 Beam 후보군이 공유할 수 있음.
- 그러나 각 후보군이 디코딩을 진행하면서 서로 다른 경로로 분기(diverge)할 수 있기 때문에, 공유 패턴이 동적으로 변화함.
- 즉, Beam Search에서는 프롬프트뿐만 아니라 중간 단계의 KV 캐시도 공유할 수 있으며, 이를 효과적으로 관리해야 함!!

![Image](https://github.com/user-attachments/assets/8d2d8570-5633-4727-a082-900dd12f22b5)

Fig. 9는 Bean Width \\(k=4\\) 인 경우, vLLM이 KV 블록을 어떻게 관리하는지 보여줍니다. 

1. 초기 프롬프트 블록 공유 (Block 0)
   - 모든 Beam 후보군(0~3)은 프롬프트를 공유하므로, Block 0을 함께 사용합니다.
   - 즉, 프롬프트의 KV 캐시는 중복 저장되지 않고, 하나의 블록만 할당되어 메모리를 절약합니다.

2. Beam 후보군이 서로 다른 경로로 분기 (Divergence)
   - Beam Candidate 3은 Block 2부터 다른 Beam과 분기되어 독자적인 KV 캐시를 사용합니다.
   - Beam Candidates 0, 1, 2는 Block 1, Block 3까지 공유하다가 Block 4에서 일부 분기됩니다.

3. 새로운 Beam 후보군이 선택되면서 불필요한 블록 해제
   - 다음 디코딩 스텝에서는 상위 4개의 후보군이 모두 Beam Candidates 1과 2에서 파생됩니다.
   - 즉, Beam Candidate 0과 3은 더 이상 높은 확률을 가지지 못하므로 제거됨.
   - 이제 남아있는 Beam은 Candidate 1과 2에서 출발하는 새로운 후보군들입니다.
   - 불필요해진 블록(Block 2, Block 4, Block 5, Block 8)은 참조 카운트가 0이 되면서 해제됩니다.

4. 새로운 KV 블록 할당 (Block 9-12)
   - vLLM은 이제 남아있는 Beam 후보군에 대해 새로운 KV 블록을 할당하여 저장합니다.
   - 이 과정에서 가능한 한 기존 블록(Block 0, 1, 3, 6, 7)을 공유하여 메모리를 효율적으로 사용합니다.

✅ <u>기존 Beam Search vs. vLLM 최적화 방식 비교</u>

|기존 Beam Search	|vLLM (PagedAttention 활용)|
|-------------|--------------|
|모든 Beam 후보군이 개별적으로 KV 캐시 저장 → 메모리 낭비 심함	|프롬프트 및 중간 블록까지 공유 가능, 필요할 때만 Copy-on-Write 적용|
|Beam 후보군이 늘어날수록 메모리 사용량이 급증	|동적으로 불필요한 KV 블록을 해제하여 메모리 사용 최적화|
|Beam Width가 커지면 GPU 메모리 제한으로 인해 병렬처리 어려움|	PagedAttention을 활용하여 더 많은 Beam을 효율적으로 관리 가능|
|모든 Beam 후보군이 끝날 때까지 메모리가 유지됨	|확률이 낮은 후보군은 조기에 제거하여 메모리를 즉시 확보 가능|


기존 LLM 서빙 시스템에서는 Beam 후보군 간에 KV 캐시를 빈번하게 복사해야 했습니다. 예를 들어, Fig. 9에서 후보군 3이 후보군 2에서 출발할 경우, 기존 KV 캐시를 복사한 후 새로운 KV 캐시를 추가해야 합니다. vLLM의 물리 블록 공유 기법을 통해 이러한 빈번한 메모리 복사 오버헤드를 크게 줄일 수 있습니다. vLLM에서는 대부분의 KV 블록이 서로 다른 Beam 후보군 간에 공유될 수 있습니다. Copy-on-Write(CoW) 기법은 새로운 토큰이 기존 공유 블록 내에서 생성될 때만 적용됩니다. 기존 Beam 후보군이 공유하고 있던 블록에서 새로운 토큰이 추가될 경우,
해당 블록을 그대로 사용할 수 없기 때문에 Copy-on-Write(CoW) 기법을 적용하여 새로운 블록을 할당합니다. 하지만, 이 과정에서 전체 KV 캐시를 복사하는 것이 아니라, 오직 하나의 블록만 복사하여 수정하면 되므로, 메모리 복사량이 최소화됩니다.





## Reference

- Kwon, Woosuk, Zhuohan Li, Siyuan Zhuang, Ying Sheng, Lianmin Zheng, Cody Hao Yu, Joseph E. Gonzalez, Hao Zhang, and Ion Stoica. “Efficient Memory Management for Large Language Model Serving with PagedAttention.” arXiv, September 12, 2023. https://doi.org/10.48550/arXiv.2309.06180.
