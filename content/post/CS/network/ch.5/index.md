+++
author = "Soeun"
title = "[네트워크] Network layer - Control plane"
date = "2023-11-06"
summary = "네트워크 레이어 - control plane 에 대한 모든 것"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "network-layer-control-plane"
series = ["Network"]
series_order = 14
+++
{{< katex >}}

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

$$cost\ of\ path(x_{1},x_{2},...,x_{p})= c(x_{1},x_{2}) + c(x_2,x_{3})+ ... + c(x_{p-1},x_{p})$$   

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

이 알고리즘의 시간복잡도를 살펴보자. n 개의 node 가 있을 때, 각 iteration 마다 루트가 정해지지 않은 모든 node 를 체크해야  한다.  따라서 시간복잡도는 \\(O(n^2)\\) 이다. 가장 효율적으로 구현한다면, \\(O(nlogn)\\) 까지 줄일 수 있다. 

###  Distance Vector Algorithm

DV algorithm 은 <span style="background-color: #FEFBD1">**iterative, asynchronous, distributed**</span> 되어 있다. 분산되어 있다는 것은, 각 노드가 직접적으로 연결된 노드로부터 정보를 받고, 계산을 하고, 이 결과를 또 자신과 연결된 노드에게 전달한다. 이 작업이 더 이상 이웃 노드 간에 교환할 정보가 없을 때까지 이루어진다. 시그널이 없으면 종료된다는 점에서 self-terminating 이라고도 한다.  비동기라는 말은, 어떠한 계산을 수행하기 위해 전체 노드들이 모두 필요하지는 않다는 뜻이다. 

여기서는 주로 <span style="background-color: #FEFBD1">**Bellman-Ford equation**</span>  을 사용한다. 
$$let\ d_{x}(y)= cost of least-cost path from x to y$$
$$then\ d_{x}(y) = min_v{c(x,v)+d_{v}(y)}$$ 

- \\(c(x,v)\\) : cost to neighbor v
- \\(d_{v}(y)\\) : cost from neighbor v to destination y

예시는 아래와 같다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/af22daae-34b1-492e-b1df-14f3ec620ef9">

- \\(D_{x}(y)\\) : estimate of least cost from x to ;y
	- x maintains distance vector \\(D_{x} = [D_{x}(y): y \in N]\\), containing x's estimate of its cost to all destinations y, in N
- node x: 
	- knows cost to each neighbor v : \\(c(x,v)\\)
	- maintains its neighbors distance vectors. For each neighbor v, x maintains \\(D_{v} = [D_{v}(y): y \in N]\\)

여기서 핵심적인 아이디어는, 노드는 자신의 distance vector estimate 를 자신의 이웃들에게 보낸다. x 가 이웃으로부터 새로운 DV estimate 를 받으면, B-F equation 을 사용하여 본인의 DV를 업데이트한다. 

$$D_{x}(y) \leftarrow \min_{v} ( c(x,v) + D_{v}(y) ) \quad \forall y \in N$$

그러면 DV의 estimate  는 결국에는 실제 least cost 인 \\(d_{x}(y)\\) 로 수렴한다. 

<img width="250" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/328a6f24-543c-4d71-978d-5bd7ec942e24">

#### DV : Link-cost changes and Link failure

> 'Good news travel fast'

link cost 가 바뀌는 상황을 생각해보자. 노드는 근접해 있는 링크의 cost 변화를 감지한다. 그 후 라우팅 정보를 업데이트하고, DV 를 재계산한다. DV 가 바뀌면, 이웃들에게 알려준다. 

<img width="250" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e32df7db-ea7e-4161-85a4-4992ef91bef8">

위 상황을 보자. 
- \\(t_0\\) : y 가 link cost 가 바뀐 것을 감지하고, DV 를 업데이트하고 이웃에게 알린다.
- \\(t_1\\) : z 가 y 로부터 업데이트 소식을 전달받고, 본인의 테이블을 업데이트 하고, x로의 least cost 를 업데이트하고 이웃들에게 DV 를 알린다.
  \\(t_2\\) : y 는 z 의 업데이트 소식을 받고, 본인의 distance table 을 업데이터한다. y 의 least cost 는 바뀌지 않는다( z -> x : 2, y->x : 1, y 는 이미 better cost 를 갖고 있기 때문에), 그래서 y 는 z 에게 더 이상 메세지를 보내지 않는다.

> 'Bad news travels slow'

다음 상황을 보자. 이번에는 x-y 사이 링크의 cost 가 4에서 60으로 훨씬 커졌다. 

<img width="250" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/aec3fd94-b299-4aca-8b4e-8385d341b337">

1. link cost 가 바뀌기 전에, \\(D_{y}(x) = 4, D_{y}(z) = 1, D_{z}(y)=1,D_{z}(x)=5\\) 였다. \\(t_0\\) 에서 y 는 link cost 변화를 감지하고, y 는 x 로의 새로운 최소 비용 경로를 계산한다. 
   $$D_{y}(x) = min ( c(y,x) + D_{x}(y), c(y,z)+D_{z}(x) ) = min ( 60+0,1+5 )=6$$ 
   당연히 여기서, z 를 통한 경로의 비용은 잘못 계산되었다. 하지만 현재 y 가 아는 정보는 옆에 link cost 가 60이 되었다는 것과, 과거에 z 가 x 로 5의 비용으로 갈 수 있다고 한 것이다. 그렇다면 y 는 z 가 x 로 5의 비용만을 가지고 갈 수 있다고 믿으면서 z 로 간다. 
   
   \\(t_1\\) 에서 <span style="background-color: #FEFBD1">**routing loop**</span>  이 발생한다.  x 로 가기 위해서는, y 는 z 로 가고, z 는 다시 y 로 간다. 이 루프는 마치 블랙홀과 같다. x 로 가기 위한 패킷은 y 와 z 사이를 왔다갔다 할 것이다. 
	
2. y 가 x 로의 새로운 경로를 계산해서, z 에게 \\(t_1\\) 의 DV 를 알려준다.
3. \\(t_1\\) 시점 이후, z 는 y 의 DV 정보를 받고, y 의 x 까지의 최소 경로 비용은 6이라는 사실을 알게 된다. z 는 y 에게 1의 비용으로 갈 수 있다는 것과, y가 x 까지 6으로 갈 수 있다는 것을 알게 된다. 따라서 z 에서 x 까지 새로운 DV는, \\(D_{z}(x) = min ( 50+0,1+6 )=7\\) 이 된다. z에서 x까지 least cost 가 업데이트 되었으므로, \\(t_{2}\\) 에이 사실을 y 에게 알려준다. 
4. 비슷하게, y는 z 의 업데이트 소식을 듣고, \\(D_{y}(z) = 8\\) 로 업데이트 한다. z 는 이 소식을 듣고 , 또 \\(D_{z}(x)=9\\) 로 업데이트 한다. 

그 과정이 언제까지 지속될까? y 와 z 사이의 루프는 z가 x로 가는 루트를 50 이상으로 계산할 때까지 44번 동안 지속될 것이다. 이 시점에서 z 는 x까지의 least cost 루트가, y 를 통해 가는 것이 아니라 직접적으로 가는 것임을 깨닫게 된다. 

#### Poisoned Reverse

위의 상황을 <span style="background-color: #FEFBD1">**poisoned reverse**</span> 를 사용해서 해결할 수 있다. z 가 x로 가기 위해 y 를 통과하면, z 는 y 에게 z -> x 비용은 무한대라고 말하는 것이다. 그렇다면 y 는 z-> x 로 가는 루트가 없다고 믿을 것이고, x에게 z를 통해 갈 시도 자체를 하지 않을 것이다. 이것은 z 가 x로 갈 때 y 를 경유하는 동안 지속된다. 

위의 상황에서 보자.
- Whenever Z routes throuth Y to get to X:
	- Z tells Y that Z->X is infinite
	- Z shifts its route to X using (Z,X) and informs that \\(D_{z}(x)=50\\) to Y
	- Then, Y sets \\(D_{y}(x)=51\\) but poisons its reverse path from Z to X, informing \\(D_{y}(x)=\infty\\)

### LS vs. DV

- **Message Complexity**
	- LS : with n nodes, E links , O(nE) msgs sent
	- DV : exchange between neighbors
- **Speed of Convergence**
	- LS : \\(O(n^2)\\) algorithm requires O(nE) msgs
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


## 4. routing among the ISPs : BGP

OSPF 는 intra-AS 프로토콜이다. 그렇다면 다른 AS 끼리 소통할 때는 어떤 프로토콜을 사용해야 할까? 바로 **<span style="background:#FEFBD1">inter-autonomous system routing protocol</span>** 이다. 소통하는 AS끼리는 같은 inter-AS 프로토콜을 사용해야 한다. 인터넷에서는 모든 AS 가 **<span style="background:#FEFBD1">BGP(Border Gateway Protocol)</span>** 이라는 같은 inter-AS 프로토콜을 사용한다. BGP 를 인터넷을 붙여주는 접착제로 표현하기도 한다. 

### The Role of BGP
BGP 의 책임을 이해하기 위해, AS 와 그 AS 안에 있는 임의의 라우터를 알아보자. 각 라우터는 forwarding table 이 있다. 만약 목적지가 같은 AS 내부에 있다면, intra-AS 프로토콜에 의해 forwarding table 이 계산된다. 그러나 목적지가 AS 바깥에 있다면 ? 이때 BGP 가 작동한다.

BGP 에서는, 패킷은 특정 목적지로 라우팅 되는 것이 아니라, CIDRized prefix 로 라우팅된다. 각 prefix 는 subnet 또는 subnet 의 집합을 가르킨다. BGP 에서, 목적지는 138.16.68/22 형태로 나타낼 수 있다. 라우터는 forwarding table 은 form (x,l) 의 entry 들을 가질 것이다. x 가 prefix 이고, l 은 라우터의 인터페이스를 가리키는 인터페이스 넘버이다. 

BGP 는 AS에 아래의 역할을 제공하는데, 
- 이웃 AS 들에게서 subnet reachability 정보를 가져온다.
- 이 정보를 AS 내에 있는 모든 라우터들에게 전달한다.
- reachability 정보 와 policy 를 기반으로 "좋은" 루트를 계산한다.

subnet 이 나머지 인터넷에게 자신의 존재를 알릴 수 있게 해준다. 

<img width="450" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c14991f4-8c7e-4259-9446-31e6413ff930">

###  Advertising BGP Route Information



<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8e2ee9dd-6e18-451b-827b-c38b10e82463">

라우터는 **gateway router** 이거나 **internal router** 이다. gateway router 는 AS 의 경계에 있어서 다른 AS 에 있는 라우터와도 연결할 수 있는 라우터이다. internal router 는 자신의 AS 내에 있는 라우터들만 연결한다. 

위의 그림에서 prefix x 를 홍보하는 상황을 보자. AS3 gateway 라우터 3a 가 AS3,X 의 path 를 AS2의 gateway 라우터인 2c 에게 홍보한다. AS3 는 AS2 에게 X 로 가는 datagram 을 포워딩 할 것이라고 약속한다. 

BGP 에서는 라우터 쌍은 반영구적인 TCP 연결을 이용하여 BGP 메세지를 주고받는다. 이것을 **<span style="background:#FEFBD1">BGP Connection</span>** 또는 **<span style="background:#FEFBD1">BGP Session</span>** 이라고 한다. 두개의 AS에 걸쳐있는 BGP Connection 을 **<span style="background:#FEFBD1">external BGP(eBGP)</span>** 라고 하고, 같은 AS 내에 있는 것을 **<span style="background:#FEFBD1">internal BGP(iBGP)</span>** 라고 한다. AS 끼리 연결하는 1개의 eBGP 가 있다. 위의 그림에서 2c 와 3a 를 연결하는 링크가 바로 eBGP 이다. AS내에서 라우터끼리 연결하는 것이 iBGP 라고 했다. 이것은 TCP 연결을 통해 이루어진다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d66a7ff8-b520-4d60-9423-57f9ae95aead">

reachability info 를 전달하기 위해, iBGP, eBGP 모두 쓰인다. x의 정보를 AS1, AS2 에 있는 모든 라우터에게 홍보한다고 하자. 이때, gateway 라우터인 3a 가 먼저 AS2 의 gateway 라우터인 2c 에게 "AS3 x" 라는 eBGP 메세지를 보낸다. 2c 는 이 메세지를 AS2 에 있는 모든 라우터들에게 iBGP 로 보낸다. 그러면 2a는 "AS2 AS3 x" 를 eBGP 를 이용해 AS1 의 1c 에게 전달한다. 3a 가 직접 1c 에게 "AS3 x" 메세지를 전달할 수도 있다. AS1 은 그러면 이 2가지 path 중에서 좋은 path를 고르고, 이 path 를 iBGP 를 통해 AS1 의 라우터들에게 전달한다. 


### Determining the Best Routes

위에서 봤듯이, 목적지로 가는 길은 여러개 일 수 있다. 실제로 인터넷에서 , 라우터는 10개 이상의 루트를 받는다. 그렇다면 여기서 라우터는 어떻게 최선의 루트를 선택할 것인가?

이 문제를 다루기 전, BGP 용어들에 대해 더 살펴보자. 홍보된 prefix 는 **<span style="background:#FEFBD1">BGP attributes</span>** 를 포함한다. 즉, prefix + attributes = "routes" 이다. 중요한 2가지 속성은 AS-PATH, NEXT-HOP 이다.

- **AS-PATH** : list of ASes through which prefix advertisement has passed
- **NEXT-HOP** : indicates specifix internal-AS router to next-hop AS

AS-PATH 값을 만드려면, prefix 가 AS 로 전달될때, AS 는 본인의 ASN 을 AS-PATH 내의 리스트에 추가한다. 위의 예시에서, AS1 에서 x 로 가는 루트는 2가지이다 : "AS2 AS3", "AS3". BGP 라우터는 이 AS-PATH 를 이용하여 홍보가 looping 되는 것을 방지한다 - 만약 라우터가 본인의 AS가 path list 에 포함되어 있다는 것을 보면 그 라우터는 홍보를 거절할 것이다. 

**NEXT-HOP** 는 *AS-PATH 를 시작하는 라우터 인터페이스의 IP 주소*이다. AS1 에서 x 로 가는 "AS2 AS3 x" 루트의 NEXT-HOP 는 2a 라우터의 IP 주소이다. "AS3 x" 의 NEXT-HOP 는 3a 이다. 

각 BGP 루트는 3가지 요소(NEXT-HOP, AS-PATH, destination prefix) 로 구성된다. NEXT-HOP이 AS1에 속하지 않는 라우터의 IP 주소임을 기억하자. 대신에, 이 IP 주소는 AS1 과 직접적으로 연결된 라우터의 IP 주소이다. 

Route selection 에 고려할 수 있는 점은 아래이다. 
- policy decision
- shortest AS-PATH
- closest NEXT-HOP router : hot potato routing
- additional criteria

#### Route Selection Algorithm

BGP 는 아래 단계들을 거치면서 루트들을 소거해가며 최선의 루트를 찾는다.

1. local preference 를 이용해서 가장 높은 값을 가진 루트들을 선정한다.
2. 가장 짧은 AS-PATH 를 가진 루트들을 선정한다.
3. hot potato routing 을 사용한다.
4. BGP identifier 를 사용한다. 


#### Hot Potato Routing

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b17b2a4d-16a4-49b6-91c5-e315a93a15fb">

가장 간단한 라우팅 알고리즘인 **<span style="background:#FEFBD1">Hot Potato Routing</span>** 에 대해 알아보자. 위의 예시에서 라우터 2d 는 x 로 가는 2가지 루트를 알게 된다. Hot Potato Routing 에서 루트는 NEXT-HOP 라우터까지 가는데 비용이 최소한인 루트로 정해진다. 2d 는 intra-AS 라우팅 정보를 이용하여 NEXT-HOP 라우터 (1c 또는 3a) 로 가는 least-cost-intra-AS path 를 구한다. 그런 후 그 중에서 최소 비용인 루트를 선택한다. 만약 1c 가 최소라고 하면, 1b 는 본인의 인터페이스에서 1c 로 가는데 비용이 최소한인 인터페이스(/)를 찾고, (x,/) 를 forwarding table 에 추가한다. 

Hot Potato Routing 의 핵심은 뒤의 cost 는 무시한 채 본인의 AS 에서 가장 최소한의 비용으로 빨리 나가는 것이다. 이름에서도 알 수 있듯이 뜨거운 감자(패킷)을 최대한 빨리 다른 사람의 손(AS) 로 옮기는 것이다. 따라서 Hot Potato Routing 는 이기적인 알고리즘이다. 본인 AS 내에서의 비용은 최소화하지만, 그 외 비용은 신경쓰지 않는다. 

#### Achieving policy via advertisements

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8064ec08-469a-44e8-a0a4-ab462458d277">

ISP 가 그의 고객 네트워크만을 이용하고 싶다고 하자. A는 Aw 를 B와 C 에게 홍보한다. 이때 B 는 BAw 를 C 에게 홍보하지 않기로 선택한다. 왜냐면 B 는 CBAw 를 라우팅함으로써 얻는 이득이 전혀 없다고 하자. C 는 그러므로 CBAw 루트에 대해 알지 못한다. 따라서 C는 무조건 CAw 를 선택할 것이다. 

A,B,C 가 provider network 이고, X,W,Y 는 손님이다. X 는 2개의 네트워크에 연결되어 있는데 이것을 **<span style="background:#FEFBD1">dual-homed</span>** 라고 한다. x는 BxC 의 라우팅을 하고 싶지 않다고 하면, x 는 B 에게 C 로 가는 루트를 홍보하지 않을 것이다. 

### Inter-AS vs. Intra-AS

|비교|Inter-AS|Intra-AS|
|---|---|---|
|policy|admin wants control over how its traffic routed, who routes through its net|single admin, so no policy decisions needed|
|performance|policy may dominate over performance|can focus on performance|

- scale :hierarchical routing saves table size, reduced update traffic


## 5. The SDN Control plane

SDN 은 **Software defined Networking** 이다. 전통적으로, 네트워크 레이어는 distributed, per-router 접근방식을 택했었다. 각각의 라우터 하나가 switching hardware, 인터넷 프로토콜을 라우터 OS 에서 실행했다. 하지만 각각 네트워크 레이어 기능들마다 다양한 middlebox 들이 생겼다. 따라서 네트워크 코어가 너무 복잡해졌다. 이것이 SDN 이 나오게 된 계기이다. 

### Characteristics of SDN

그렇다면 왜 logically centralized control plane 을 도입했을까? 아래는 4가지 특징이다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Flow-based forwarding**</span></span>  

SDN 에 의해 제어되는 패킷 포워딩은 transport-layer, network-layer, link-layer 헤더 값 중 어느 것에도 의존할 수 있다. OpenFlow 1.0 abstraction 은 포워딩이 11개의 다른 헤더 필드 값에 의존할 수 있게 했다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Seperation of data plane and control plane**</span></span>  

라우터는 오직 "match plus action" 에 집중할 수 있게 되었다. 따라서 더 빠르고 단순하게 작동할 수 있게 되었다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Network Control functions : external to data-plane switches**</span></span>  

control plane 은 2가지 요소(SDN controller, set of network-control applications) 로 이루어진다. 컨트롤러는 logically centralized 되어 있다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Programmable network**</span></span>  

Software 에 의해 제어된다는 것은 SDN controller 가 "programmable" 해진 것이다. 따라서 네트워크 알고리즘을 업데이트 할 수 있다. 

#### Traffic engineering

전통적인 라우팅에서, traffic engineering 이 쉽지 않았다. 만약 link cost 가 바뀌지 않아도 루트를 바꾸려면, 라우터 자체를 바꿔야 했다. 하지만 SDN 하에서, 네트워크 경로를 바꾸는 것은 훨씬 쉬워졌다. 
 
### SDN Controller and SDN Network-control Applications

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cdbf60ea-2475-4aad-bfb2-e4e8db77920c">


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Data plane switches**</span></span>  

data plane 에 있는 스위치들은 빠르고, 간단하고, 포워딩을 한다. swith flow table 은 컨트롤러에 의해 계산된다. table-based swith control 을 위한 API 가 있다.  컨트롤러와 소통을 하는데, 이때 사용하는 프로토콜이 OpenFlow 이다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**SDN Controller(network OS)**</span></span>  

중간 레이어에서는 northbound API 를 이용해 네트워크 컨트롤 어플리케이션과 소통한다. 아래에 있는 네트워크 스위치와는 southbound API 를 이용해 소통한다. 성능을 위해 물리적으로 분산되어 구현되어 있다.

디바이스는 로컬에서 일어난 일들을 컨트롤러에게 전달해주어야 한다. 따라서 SDN controller 는 항상 네트워크 상태에 대한 최신 정보를 유지한다. 컨트롤로와 디바이스 간의 소통은 "southbound" interface 로 불린다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Control Applications**</span></span>  

컨트롤러의 뇌 역할을 한다. 아래 레이어 서비스들을 이용해 control functions 를 구현한다.


아래는 SDN Controller 를 이루는 구성요소이다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/498da93b-b2ea-4e15-80de-6ad896a4be37">


### OpenFlow Protocol

OpenFlow 프로토콜은 controller 과 switch 사이에 작동한다. 메세지를 교환하기 위해 TCP 를 사용한다(port num 6653). 

3가지 클래스가 있다. 
- controller-to-switch
- asynchronous
- symmetric

#### controller -> switch
- features
- configure
- modify-state
- packet-out

#### switch -> controller
- packet-in
- flow-removed
- port status

예시를 보자.

<img width="400" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b294bcbc-f660-460a-862e-80441b635c81">


1. ﻿﻿S1, experiencing link failure using OpenFlow port status message to notify controller
2. ﻿﻿SDN controller receives  OpenFlow message, updates link status info
3. ﻿﻿Dijkstra's routing algorithm application has previously registered to be called when ever link status changes. It is called.
4. ﻿﻿Dijkstra's routing algorithm access network graph info, link state info in controller, computes new routes
5. link state routing app interacts with flow-table-computation component in SDN controller, which computes new flow tables needed
6. Controller uses OpenFlow to install new tables in switches that need updating

- 실제 컨트롤러 : OpenDaylight controller, ONOS controller 

## 6. ICMP : The Internet Control Message Protocol

ICMP 는 네트워크 레벨 정보를 소통하기 위해 호스트와 라우터가 사용한다. 에러를 보고하거나, 요청/답변을 echo 하는데 사용할 수 있다. 예시로, HTTP 요청을 보냈을 때 "Destination network unreachable" 이라는 에러 메세지를 본 적이 있을 것이다. IP 라우터가 어느 순간 HTTP 요청에 써있는 호스트를 찾지못했을 것이다. 그러면 라우터는 ICMP  메세지를 만들어서 보낸다. 

ICMP 는 IP 에 속하는 것으로 느껴지지만, 실제로는 IP 위에 있다. ICMP 메세지들은 IP datagram 안에 전달된다. 즉, TCP / UDP segment 가 IP datagram 안에 payload 로 전달되는 것처럼 ICMP 메세지도 IP payload 로 전달된다. 

ICMP 메세지는 **type** 과 **code** field 가 있다. 

**Traceroute** 를 알기 위해 ICMP 를 활용할 수 있다. source 는 목적지에게 UDP segment들을 연속적으로 보낸다. 여기에는 unlikely port number 가 목적지이다.  첫번째 set 는 TTL = 1, 두번째는 TTL = 2 ... 또 souce 는 datagram 을 보낼 때 타이머를 시작한다. datagram 이 n번째 라우터에 도착했을 때, 라우터는 datagram 을 버리고 source 에게 ICMP 메세지를 보낸다(type 11, code 0). ICMP 메세지는 라우터의 이름과 IP 주소를 포함한다. ICMP 메세지가 도착하면 source 는 RTT 를 기록한다. 

UCP segment 가 결과적으로 목적지 host 에 도착하면, 목적지는 "port unreachable" 이라는 ICMP 메세지(type 3, code 3) 을 보낸다. 그러면 source 도 멈춘다. 

## Reference
- Computer Networking A Top Down Approach , 7th edition, ch.5
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 