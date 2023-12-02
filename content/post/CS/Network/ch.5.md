+++
author = "Soeun"
title = "[네트워크] Network layer - Control plane"
date = "2023-11-06"
description = "네트워크 레이어 - control plane 에 대한 모든 것"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "network-layer-control-plane"
+++

## 1. Introduction

네트워크 레이어의 data , control plane 에서 핵심적인 것은 forwarding table(destination based forwarding), flow table(generalized forwarding) 이었다. 

이번 챕터에서는 어떻게 forwarding, flow table 이 계산되고 유지되고 설치되는지 알아볼 것이다. Network control plane 을 접근하는 2가지 방법에는 per-router control, logically centralized control 이 있다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Per-router control**</span></span> 

각 라우터 내부에서 라우팅 알고리즘이 계산된다. 즉 라우터 안에서 forwarding, rounting 기능이 다 있다. 전통적인 방법이다. 뒤에서 볼 OSPF 와 BGP 프로토콜이 여기에 기반을 두고 있다.

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Logically centralized control**</span></span> 

중앙 집중된 라우팅 컨트롤러에서 forwarding table 을 계산한다. 중앙에 있는 컨트롤러는 라우터 내부의 Control Agent(CA) 와 소통해서 flow table 을 관리한다. CA 는 정말 최소한의 기능만 있고, CA 끼리는 소통하지 않는다.  

컨트롤러가 중앙 집중화 되어 있다고 해서 물리적으로 하나만 존재하는 것은 아니다 ! Virtually centralized 되어 있는 개념이다. 

## 2. Routing Protocols

**라우팅 알고리즘**의 목적은, 발신 호스트에서 수신 호스트까지 라우터의 네트워크 내부에서 좋은 루트 (성능이 좋은 것, 가장 빠른.., 가장 짧은, 비용이 적게 드는) 를 찾는 것이다. 하지만, 실제 세계에서는 policy issue 도 존재한다. 예를 들어, 라우터x 가 Y 회사에 속하고, 이것은 회사Z에 속하는 네트워크로 패킷을 전송하면 안된다 와 같은 규칙이 존재할 수 있다. 

라우팅 문제를 풀 때 주로 그래프를 활용한다. 

<img width="400" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8dfdfebd-8723-4211-aa20-386601e8cf1f">

- graph : G = (N,E)
- N = set of routers = {u,v,w,x,y,z}
- E = set of links ={ (u,v), (u,x), (v,x), (v,W), (x,W), (X,Y), (W,Y), (W,Z), (y,z) }

여기서 cost 를 계산해보자. cost 는 bandwidth 와 반비례해야 하고, congestion 과도 반비례한다.

$cost\ of\ path(x_{1},x_{2},...,x_{p})= c(x_{1},x_{2}) + c(x_2,x_{3})+ ... + c(x_{p-1},x_{p})$  

가장 중요한 문제는 아래 2개이다. 
- **u 와 z 사이에 least-cost path 가 어디인가?**
- **어떤 라우팅 알고리즘을 사용해서 계산할 것인가?** 

알고리즘은 분류하는 방식에는 크게 3가지가 있다. 
1. **centralized, decentralized**
   
   만약 cost 를 계산할 때 전체 path 의 각 cost 를 알고 계산한다면 그것은 **centralized** 방식이다. 반면, 모든 길의 cost 를 다 알지 못한다면 그것은 **decentralized** 방식이다. 

- **Centralized routing algorithm (global)**

    네트워크에 대한 global 지식이 있는 상태로 least-cost path 를 계산한다. 계산은 하나의 사이트(e.g. logically centralized controller) 에서 하거나, 각 라우터에서도 할 수 있다. 이 알고리즘을 **<span style="background:#FEFBD1">Link-state algorithm</span>** 이라고 한다. 

- **Decentralized routing algorithm**

    least-cost 에 대한 계산은 각 라우터 내부에서 이루어진다. 각 노드는 전체 cost 에 대한 정보가 없고, 단순히 직접적으로 연결된 링크의 cost 만 안다. iter 를 돌면서 반복적으로 이웃 node 들과 정보를 교환한다. 이것을 **<span style="background:#FEFBD1">distance-vector(DV) algorithm</span>** 이라고 한다.

2. **static / dynamic** 
   
    static routing algorithm 은 천천히 업데이트 된다. Dynamic routing algorithm 은 network traffic load 또는 topology 가 변화할 때마다 루트를 업데이트 한다. 

3. **load-sensitive / load-intensive**

    load-sensitive algorithm은 현재 링크의 congestion 상태를 반영하기 위해 크게 변동된다. 만약 링크가 혼잡하다면, 이 알고리즘은 다른 루트를 선택한다. 반면 load-intensive algorithm 은 링크의 congestion 상태를 반영하지 않는다. 

### Link-State Routing Algorithm

link-state algorithm은 네트워크 내에 있는 모든 링크의 cost 를 알고 시작한다. 실제로는, 각 노드가 link-state 패킷을 전체 다른 노드들로 broadcast 한다. 각 link-state 패킷은 붙어있는 링크의 identity 와 cost 에 대한 정보를 담고 있다. 이것을 **<span style="background:#FEFBD1">link-state broadcast</span>** 라고 한다. 이 결과로 모든 노드는 동일하게 네트워크 전체에 대한 정보를 가지고 있다. 

link-state algorithm은 **<span style="background:#FEFBD1">Dijkstra's algorithm</span>** 을 사용한다. 이 알고리즘은 하나의 노드에서 네트워크 내 다른 노드들까지의 최소 비용 경로를 계산한다. 알고리즘의 kth iteration 후에는, k 목적지 노드까지의 최소 경로 비용을 알 수 있다. 

이 알고리즘을 정의해보자. 
- c(x,y) : link cost from node x to y. If not direct, set to infinite
- D(v) : current value of cost of path from souce to dest. v
- p(v) : predecessor node along path from source to v
- N' : set of nodes whose leash cost path definitively known

pseudo code 는 아래와 같다.
```text
Initialization:

N' = {u}
for all nodes v
	if v adjacent to u
		then D(v) = c(u,v)
	else D(V) = ∞

Loop

find w not in N' such that D(w) is a minimum
	add w to N'
	update D(v) for all v adjacent to w and not in N' :
		D(v) = min(D(v), D(w) + c(w,v))

/* new cost to v is either old cost to v or known
shortest path cost to w plus cost from w to v */ 

until all nodes in N'
```

예시는 아래와 같다.

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/34bc8569-c685-4b0a-849f-8b091c11c702">

이 알고리즘의 시간복잡도를 살펴보자. n 개의 node 가 있을 때, 각 iteration 마다 루트가 정해지지 않은 모든 node 를 체크해야  한다.  따라서 시간복잡도는 $O(n^2)$ 이다. 가장 효율적으로 구현한다면, $O(nlogn)$ 까지 줄일 수 있다. 

###  Distance Vector Algorithm

DV algorithm 은 <span style="background-color: #FEFBD1">**iterative, asynchronous, distributed**</span> 되어 있다. 분산되어 있다는 것은, 각 노드가 직접적으로 연결된 노드로부터 정보를 받고, 계산을 하고, 이 결과를 또 자신과 연결된 노드에게 전달한다. 이 작업이 더 이상 이웃 노드 간에 교환할 정보가 없을 때까지 이루어진다. 시그널이 없으면 종료된다는 점에서 self-terminating 이라고도 한다.  비동기라는 말은, 어떠한 계산을 수행하기 위해 전체 노드들이 모두 필요하지는 않다는 뜻이다. 

여기서는 주로 <span style="background-color: #FEFBD1">**Bellman-Ford equation**</span>  을 사용한다. 
$let\ d_{x}(y)= cost of least-cost path from x to y$ 
$then\ d_{x}(y) = min_v{c(x,v)+d_{v}(y)}$ 

- $c(x,v)$ : cost to neighbor v
- $d_{v}(y)$ : cost from neighbor v to destination y

예시는 아래와 같다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/af22daae-34b1-492e-b1df-14f3ec620ef9">

- $D_{x}(y)$ : estimate of least cost from x to ;y
	- x maintains distance vector $D_{x} = [D_{x}(y): y \in N]$, containing x's estimate of its cost to all destinations y, in N
- node x: 
	- knows cost to each neighbor v : $c(x,v)$
	- maintains its neighbors distance vectors. For each neighbor v, x maintains $D_{v} = [D_{v}(y): y \in N]$

여기서 핵심적인 아이디어는, 노드는 자신의 distance vector estimate 를 자신의 이웃들에게 보낸다. x 가 이웃으로부터 새로운 DV estimate 를 받으면, B-F equation 을 사용하여 본인의 DV를 업데이트한다. 

$D_{x}(y) \leftarrow \min_{v} ( c(x,v) + D_{v}(y) ) \quad \forall y \in N$ 

그러면 DV의 estimate  는 결국에는 실제 least cost 인 $d_{x}(y)$ 로 수렴한다. 

<img width="250" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/328a6f24-543c-4d71-978d-5bd7ec942e24">

#### DV : Link-cost changes and Link failure

> 'Good news travel fast'

link cost 가 바뀌는 상황을 생각해보자. 노드는 근접해 있는 링크의 cost 변화를 감지한다. 그 후 라우팅 정보를 업데이트하고, DV 를 재계산한다. DV 가 바뀌면, 이웃들에게 알려준다. 

<img width="250" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e32df7db-ea7e-4161-85a4-4992ef91bef8">

위 상황을 보자. 
- $t_0$ : y 가 link cost 가 바뀐 것을 감지하고, DV 를 업데이트하고 이웃에게 알린다.
- $t_1$ : z 가 y 로부터 업데이트 소식을 전달받고, 본인의 테이블을 업데이트 하고, x로의 least cost 를 업데이트하고 이웃들에게 DV 를 알린다.
- $t_2$ : y 는 z 의 업데이트 소식을 받고, 본인의 distance table 을 업데이터한다. y 의 least cost 는 바뀌지 않는다( z -> x : 2, y->x : 1, y 는 이미 better cost 를 갖고 있기 때문에), 그래서 y 는 z 에게 더 이상 메세지를 보내지 않는다.

> 'Bad news travels slow'

다음 상황을 보자. 이번에는 x-y 사이 링크의 cost 가 4에서 60으로 훨씬 커졌다. 

<img width="250" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/aec3fd94-b299-4aca-8b4e-8385d341b337">

1. link cost 가 바뀌기 전에, $D_{y}(x) = 4, D_{y}(z) = 1, D_{z}(y)=1,D_{z}(x)=5$ 였다. $t_0$ 에서 y 는 link cost 변화를 감지하고, y 는 x 로의 새로운 최소 비용 경로를 계산한다. 
   $D_{y}(x) = min ( c(y,x) + D_{x}(y), c(y,z)+D_{z}(x) ) = min ( 60+0,1+5 )=6$ 
   당연히 여기서, z 를 통한 경로의 비용은 잘못 계산되었다. 하지만 현재 y 가 아는 정보는 옆에 link cost 가 60이 되었다는 것과, 과거에 z 가 x 로 5의 비용으로 갈 수 있다고 한 것이다. 그렇다면 y 는 z 가 x 로 5의 비용만을 가지고 갈 수 있다고 믿으면서 z 로 간다. 
   
   $t_1$ 에서<span style="background-color: #FEFBD1">**routing loop**</span>  이 발생한다.  x 로 가기 위해서는, y 는 z 로 가고, z 는 다시 y 로 간다. 이 루프는 마치 블랙홀과 같다. x 로 가기 위한 패킷은 y 와 z 사이를 왔다갔다 할 것이다. 
	
2. y 가 x 로의 새로운 경로를 계산해서, z 에게 $t_1$ 의 DV 를 알려준다.
3. $t_1$ 시점 이후, z 는 y 의 DV 정보를 받고, y 의 x 까지의 최소 경로 비용은 6이라는 사실을 알게 된다. z 는 y 에게 1의 비용으로 갈 수 있다는 것과, y가 x 까지 6으로 갈 수 있다는 것을 알게 된다. 따라서 z 에서 x 까지 새로운 DV는, $D_{z}(x) = min ( 50+0,1+6 )=7$ 이 된다. z에서 x까지 least cost 가 업데이트 되었으므로, $t_{2}$ 에이 사실을 y 에게 알려준다. 
4. 비슷하게, y는 z 의 업데이트 소식을 듣고, $D_{y}(z) = 8$ 로 업데이트 한다. z 는 이 소식을 듣고 , 또 $D_{z}(x)=9$ 로 업데이트 한다. 

그 과정이 언제까지 지속될까? y 와 z 사이의 루프는 z가 x로 가는 루트를 50 이상으로 계산할 때까지 44번 동안 지속될 것이다. 이 시점에서 z 는 x까지의 least cost 루트가, y 를 통해 가는 것이 아니라 직접적으로 가는 것임을 깨닫게 된다. 

#### Poisoned Reverse

위의 상황을 <span style="background-color: #FEFBD1">**poisoned reverse**</span> 를 사용해서 해결할 수 있다. z 가 x로 가기 위해 y 를 통과하면, z 는 y 에게 z -> x 비용은 무한대라고 말하는 것이다. 그렇다면 y 는 z-> x 로 가는 루트가 없다고 믿을 것이고, x에게 z를 통해 갈 시도 자체를 하지 않을 것이다. 이것은 z 가 x로 갈 때 y 를 경유하는 동안 지속된다. 

위의 상황에서 보자.
- Whenever Z routes throuth Y to get to X:
	- Z tells Y that Z->X is infinite
	- Z shifts its route to X using (Z,X) and informs that $D_{z}(x)=50$ to Y
	- Then, Y sets $D_{y}(x)=51$ but poisons its reverse path from Z to X, informing $D_{y}(x)=\infty$

### LS vs. DV

- **Message Complexity**
	- LS : with n nodes, E links , O(nE) msgs sent
	- DV : exchange between neighbors
- **Speed of Convergence**
	- LS : $O(n^2)$ algorithm requires O(nE) msgs
	- DV : convergence time varies
		- count-to-infinity problems
- **robustness** : 라우터가 고장난다면?
	- LS : node can advertise incorrect link cost, each node computes only its own table
	- DV : node can advertise incorrect path cost, each node's table used by others

## 3. intra-AS routing in the Internet : OSPF

지금까지 우리는 라우터에 대해 너무 이상적으로 공부했다. 라우터는 모두 동일하다고 가정했지만, 실제로는 사실이 아니다. 하지만 이렇게 가정하는데는 2가지 이유가 있다.

- **Scale**
  
  수백만 가지의 목적지가 있는데, 이것을 모두 라우팅 테이블에 저장할 수는 없다. 이렇게 하면 라우터의 연결 정보와 link cost 를 브로드캐스팅하는 과정에서 링크가 마비될 것이다. 
  
- **administrative autonomy**
  
  인터넷은 ISP 들의 네트워크이다. 각 ISP 는 각자 라우터들의 네트워크로 이루어져 있다. 각 ISP 는 각자의 입맛에 맞게 네트워크를 운용하고자 할 수 있다. 

이 2가지 문제는 라우터들을 **<span style="background:#FEFBD1">autonomous systems(ASs)</span>** 로 정리하면 해결할 수 있다. ISP 내부의 라우터와 그들을 연결하는 링크는 하나의 AS 를 구성한다. 

같은 AS 내에 있는 라우터들이 모두 같은 라우팅 알고리즘 운용하고, 각각에 대한 정보를 가지고 있는 시스템을 **<span style="background:#FEFBD1">intra-autonomous system</span>** 이라고 한다. 다른 AS 에 있는 라우터와는 라우팅 알고리즘이 다를 수 있다. AS의 경계에는 gateway router 가 있어서, 다른 AS 의 라우터와 연결한다. 

**<span style="background:#FEFBD1">inter-AS routing</span>** 은 AS 끼리 연결이다. gateway 가 inter-domain routing 의 역할을 수행한다. 

<img width="450" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1e173544-b12a-438a-a703-92e6aa427805">

forwarding table 은 intra-AS, inter-AS 둘 다의 영향을 받느다. intra-AS 라우팅은 AS 내부에 목적지가 있는 entry 들을 담당한다. inter-AS & intra-AS 모두 바깥 목적이에 대한 entry 를 담당한다.

### Inter-AS Routing

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4f29d4fc-2b38-4630-8173-fed0840a8d77">

AS1 에 있는 라우터가 AS1 바깥에 있는 라우터에게 datagram 을 받는다고 하자. 라우터는 이 패킷을 gateway 라우터에게 보내야 하지만, 어느 것이 gateway 라우터일까?

AS1 은 목적지 라우터가 AS2, AS3 에서 올 때 각각 어느 라우터를 통해서 올 수 있는지 알아야 한다. 그리고 이 정보를 AS1 내에 있는 모든 라우터에게 전달해야 한다. 

예를 들어, AS2 에서 AS1 로 들어올 때 gateway 라우터는 1b 이다. 

### Intra-AS Routing

**<span style="background:#FEFBD1">interior gateway protocols(IGP)</span>** 라고도 불린다. 가장 흔한 intra-AS protocol는 아래이다. 
- RIP : Routing Information Protocol
- OSPF : Open Shortest Path First
- IGRP : Interior Gateway Routing Protocol 


### Open Shortest Path First(OSPF)
OSPF 는 link-state protocol로, link-state information 과 Dijkstra's least cost path 알고리즘을 사용한다. OSPF 를 사용해서, 각 라우터는 AS 의 topological map 을 구축한다. 각 라우터는 Dijkstra's shortest path algorithm 을 사용하여 각 subnet 에게 도달하는 가장 최적의 path tree 를 계산한다. 각 link cost 는 네트워크 관리자에 의해 결정된다. 

라우터는 OSPF link-state advertisements 를 AS 내에 있는 모든 라우터에게 전달한다. 라우터는 link state 의 변화가 있을 때마다 브로드캐스팅 하고, 변화가 없어도 주기적으로 브로드캐스팅한다. 이것은 OSPF 메세지에 포함되어, IP 에 의해 전달된다. 따라서 OSPF 프로토콜은 신뢰할 수 있는 메세지 주고받기 기능과, link-state 브로드캐스트 기능을 충분하게 수행해야 한다. 

OSPF 가 포함하는 advance features 는 다음과 같다.
- **security**
  모든 OSPF 메세지는 authenticated 된다. 
  
- **Multiple same-cost paths**
  
  같은 비용을 가지는 여러 path 를 넣을 수 있다. 
  
- **Integrated support for unicast and multicast routing**
  
  Multicast OSPF(MOSPF) 는 존재하는 기존의 OSPF link database 를 사용한다.
  
- **Support for hierarchy within a single AS**
  
  2가지 계층이 있다. local area, backbone 
  
  backbone의 역할은, AS 내부의 교통정리이다. 이 backbone 은 area-border 라우터들을 모두 포함한다. AS 내부의 Inter-routing 은 우선 패킷이 area-border 라우터로 간 뒤, backbone으로 라우팅된 후, backbone 내에서 그 패킷의 목적지를 포함하는 border 라우터로 보내진 뒤, 최종 목적지로 간다. 
  

	<img width="450" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/40b02e5d-efa3-4485-b697-15f1cadd2158">

