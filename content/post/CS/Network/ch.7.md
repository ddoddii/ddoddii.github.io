+++
author = "Soeun"
title = "[네트워크] Wireless and Mobile Networks"
date = "2023-12-01"
description = "무선 네트워크에 대한 모든 것"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "wireless-and-mobile-networks"
+++

## 1. Introduction

Wireless network 의 구성요소는 아래오 같다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/51264176-47fa-4f55-b6dc-412365fb720f">

- **wireless host**
  
  호스트는 end-system 디바이스로, 어플리케이션을 실행한다. wireless host 에는 노트북, 스마트폰 등이 있다. 정적일 수 있고, 이동할 수 도 있다(핸드폰).

- **Base station**
  
  유선 네트워크에 연결되어 있다. 그 지역내에서 유선 네트워크와 무선 호스트 간 패킷을 전송하는 책임이 있다. 예시로는 cell tower, 802.11 access point 등이 있따.

- **wireless links**
  
  무선 호스트들을 base station 에 연결할 때 사용한다. backbone link 로도 사용된다. 다른 wireless 기술 마다 전송률과 전송할 수 있는 거리가 다르다. 아래는 유명한 무선 기술마다의 전송률, 전송할 수 있는 거리를 나타낸다.
  
  <img width="450" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/995a991e-81b0-425c-95df-43db53902939">
  
- **Infrastructure mode** 
  
  Base station 이 모바일 디바이스를 유선 네트워크에 연결한다. 모바일이 이동해서 접속 지점이 바뀌는 것을 handoff 라고 하는데, 접속 지점이 바뀔때는 handoff management 를 해야 한다. 
  
- **ad hoc mode**
  
  Base station 이 없다. 노드는 link coverage 안에 있는 다른 노드들만 소통할 수 있다. 노드들은 자기들끼리 라우팅을 한다. 

무선 네트워크의 부분들을 살펴봤는데, 이 조각들을 조합해서 다른 유형의 유선 네트워크를 만들 수 있다. 2가지 기준으로 나눌 수 있다:  (1) 무선 네트워크 안의 패킷이 하나의 wireless hop 을 지나는지 / 다수의 wireless hop 을 지나는지 , (2) 네트워크 안에 base station 과 같은 infrastructure 가 있는지. 

- **Single-hop, infrastructure-based**
  
  이 네트워크는 더 큰 유선 네트워크에 연결되어 있는 base station 이 있다. 이 base station과 무선 호스트 간의 소통도 하나의 무선 hop 를 통해 이루어진다. 교실에서 쓰는 802.11 네트워크, 4G LTE 가 여기 속한다. 
  
- **Single-hop, infrastructure-less**
  
  여기에는 base station 이 없다. 대신, 네트워크 안에 있는 노드가 다른 노드들의 전송을 모두 컨트롤한다. Bluetooth, 802.11 network in ad hoc mode 이 여기 속한다. 

- **Multi-hop, infrastructure-based**
  
  base station이 있지만, 몇개의 무선 노드는 다른 노드들을 통과해야 base station과 소통할 수 있다. wireless mesh network 가 여기 속한다. 

- **Multi-hop, infrastructure-less**
  
  base station 이 없고, 무선 노드는 다른 노드들을 통과해야 base station과 소통할 수 있다. 노드들은 또 움직일 수 있다 - 노드끼리의 연결성이 바뀐다(**mobile ad hoc networks (MANETs)**). 모바일 노드가 vehicles 라면, 네트워크는 **vehicular ad hoc network (VANET)** 이다. 
  
<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f83c80f2-9d14-44da-8f12-f27138ec5317">


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Ad-hoc mode : MANET**</span></span>  

Mobile Ad-hoc network 이다. 전통적인 라우팅인 LS / DV 는 사용할 수 없다. 네트워크 간에 multi-hop path 를 찾아야 한다. 


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Ad-hoc mode : VANET**</span></span>  

VANET 은 802.11p 이다. Internet of Vehicles(IoV) 로 진화했다. MANET 에 비교해서 훨씬 기동성이 좋다. 지리적으로 라우팅할때 도로 상황을 고려해야 한다. 도시/ 산간 지역이 다르다. 


## 2. Wireless links and Network Characteristics 

집 네트워크를 유선(이더넷)에서 무선(wireless 802.11)으로 교체한다고 해보자. 바뀌는 부분은, 호스트의 유선 이더넷 인터페이스가 무선 네트워크 인터페이스으로 바뀔 것이고, 이더넷 스위치가 access point 로 바뀔 것이다. 그 외 네트워크 레이어에는 바뀌는 부분이 없다. 따라서 링크 레이어를 집중적으로 보자. 

유선이 무선 링크로 바뀌면, 차이점은 다음과 같다.
- **decreased signal strength**
  
  라디오 시그널은 물질을 통과하면 감쇠된다. 따라서 전송할 수 있는 범위가 한정된다. 
  
- **interference from other sources**
  
  같은 주파수로 전송하는 라디오 시그널은 서로에게 간섭한다. 예를들어, 2.4GHz 무선 핸드폰과 802.11b 무선 LAN 은 같은 주파수 범위를 사용한다. 또, 근처에 있는 디바이스들과도 간섭이 될 수 있다.
  
- **multipath propagation**
  
  라디오 시그널이 물체 또는 바닥에 반사되어서 목적지에 약간의 시간차를 두고 도착할 수 있다.

이러한 점들이 무선 소통을 더 어렵게 만든다. 이런 점들을 봤을때 무선 소통이 유선 소통보다 bit error 가 더 많다는 것을 짐작할 수 있다. 따라서 무선 링크 프로토콜은 더욱 강력한 CRC error detection code 와 link-level 에서 신뢰할수 있는 데이터 전송 프로토콜을 가지고 있다. 

노이즈와 같은 방해 요소 때문에 받을 수 있는 data rate 이 제한될 수 있다. **<span style="background:#FEFBD1">Channel Capacity</span>** 는 주어진 소통 경로에서 데이터가 전송될 수 있는 최대 속도이다. Bit Error Rate(BER) 는 보내진 bit 이 받는 쪽에서 에러가 될 확률이다. 이것은  **<span style="background:#FEFBD1">Signal-to-Noise Ratio(SNR)</span>** 에 좌우된다. 
 
**Signal-to-Noise Ratio (SNR)**

SNR 은 받은 전체 데이터 중 실제 정보(시그널)  과 노이즈의 비율이다. 받는 쪽에서 데시벨(dB)로 측정한다. SNR 이 높을 수록 실제 정보가 더 많으니까, 더 품질이좋은 것이다. SNR 에 따라 채널의 capacity 가 결정된다. 

$SNR_{(dB)}=10\log_{10}\frac{signal \ power}{noise \ power}$ 

Capacity 는 Shannon capacity formula 를 통해 구할 수 있다. 이것은 채널이 이론적으로 받을 수 있는 최대 대역폭 값이다. 

$C=B\log_{2}(1+SNR)$

<img width="310" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a81492ee-5286-48c4-9c1a-5ce27a19b446">

위는 BER과 SNR 의 관계를 나타낸다. SNR이 클수록, 노이즈로부터 시그널을 추출하기 쉬워진다. 하지만 여기서 tradeoff 관계가 있다. 

- **modulation scheme 이 주어졌을때, SNR이 커지면 BER 이 작아진다.** 

  보내는 쪽이 파워를 증가시켜서 SNR 을 증가시킬 수 있으므로, 이때 BER은 작아진다. 하지만 임계값 이후로 파워를 증가시키는 것은 작은 효용이 있다. 그리고 전송 파워를 증가시키는 것은 단점도 있다. 보내는 쪽에서 훨씬 더 많은 에너지가 들어간다. 그리고 다른 보내는 이의 신호에 간섭할 확률도 커진다. 

- **SNR 이 주어졌을 때, 더 큰 전송률을 가진 modulation technique 는 더 큰 BER 를 갖는다.** 
  
  예를 들어, SNR 이 10bB 일떄, BPSK는 $10^{-7}$ 보다 작은 BER 를 갖는다. 반면에 QAM16 의 BER 는 대략 $10^{-1}$ 이다. SNR이 20bB 일 때, QAM16 은 BER $10^{-7}$ 이고, BPSK 는 훨씬 작은 BER 를 갖는다. 따라서 SNR 이 주어지면, 더 큰 전송률을 가진 modulation 일 수록 더 큰 BER 를 갖는 것을 알 수 있다. 
  
- **물리적 레이어 modulation 은 채널 컨디션에 따라 다이나믹하게 선택될 수 있다.** 
  
  SNR 는 움직이거나, 환경이 변하면 변동될 수 있다. 따라서 802.11 WiFi 와 4G cellular data network 에는 adaptive modulation 이 쓰인다. 


다수의 무선 보내는 이와 받는 이는 추가적으로 아래와 같은 문제들에 직면한다. 
- **Hidden terminal problem** 
  
  <img width="263" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/65122650-62c3-4206-b083-4b3673852391">
  
  B,A 는 서로와 소통할 수 있고, B,C 도 소통할 수 있지만, A,C 는 서로 소통할 수 없다. 이 말은 A,C 가 B의 간섭에 대해 모른다는 얘기이다. 

- **Signal attenuation** 
  
  <img width="311" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/afb07f13-881c-481f-b36f-5272063b7ae4">
  
  시그널이 거리가 멀어지거나 물리적인 장애물을 만나면 감쇠한다. (B,A), (B,C) 는 소통하지만, A,C 는 시그널이 감쇠되어서 소통하지 못한다. 

### Code Division Multiple Access(CDMA)

앞서, 호스트들이 하나의 미디엄을 공유하면, 다수의 보내는 시그널들이 받는 쪽에서 서로 간섭하지 못하도록 프로토콜이 필요하다고 했다. 여기서는 무선 네트워크에서 많이 쓰이는, 각 유저에게 유니크한 code를 할당하는 CDMA 에 대해 알아보자. 

CDMA 에서 보내지는 각 bit 는 chipping rate 로 바뀌는 시그널을 곱함으로써 인코딩 된다. 모든 유저는 주파수를 공유하지만, 각 유저는 자신만의 chipping sequence 가 있다. 이것은 다수의 사용자가 공존하고, 최소의 간섭으로 전송할 수 있게 해준다. 

encoded signal = (original data) X (chipping sequence)

디코딩 할땐, encoded signal과 chipping sequence의 내적을 한다. 
<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6b17b095-931c-4166-9fc5-75ece6b811f5">

위에는 간단한 CDMA encoding/decoding 시나리오이다. 원본 데이터 bit 가 CDMA 인코더에 도착하는 시간을 unit time 이라고 가정하다. 즉, 원본 데이터 bit가 전송되는데 걸리는 시간이 one-bit slot time 이다. $d_{i}$ 가 ith bit slot 의 데이터 값 이라고 하자. 각 bit slot 는 M mini-slots 로 나줘진다. 위에서 M=8 이다. 

보내는 쪽에서 사용하는 CDMA code 는 M개 값의 연속이다. 위의 예시에서 보내는 쪽이 사용하는 코드는 (1,1,1,-1,1,-1,-1,-1) 이다. 

이제 ith data bit 인 $d_i$ 를 보자. 인코더의 output $Z_{i,m}$ 은 $d_i$ 와 CDMA code 의 mth bit $c_m$ 을 곱한 값이다. 

$Z_{i,m}=d_{i}*c_{m}$ 

방해하는 사람이 없다면, 받는 쪽은 $Z_{i,m}$ 을 온전히 받고, $d_i$ 값을 다시 디코딩할 것이다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6040f569-57e7-4447-8866-b41bdfb5259d">

하지만 CDMA 는 **방해하는 환경** 속에서도 잘 작동해야 한다. 만약 data bits 들이 모두 섞여있는 상태에서, 받는 쪽은 어떻게 보내는 쪽의 data bit 을 온전히 디코딩 할 것인가? CDMA 는 간섭하는 bit signal 이 additive 하다고 가정한다. 예를 들어, 같은 mini-slot 시간대에 3명이 1을 보내고 4번째가 -1 을 보내면 받는 쪽은 그 mini-slot 동안 받은 시그널이 (1+1+1-1 = 2) 라고 생각한다. 

위쪽의 보내는 이는 코드 (1,1,1,-1,1,-1,-1,-1)을 사용하고, 아래는 코드 (1,-1,1,1,1,-1,1,1)을 사용한다. 간섭이 있다면 sender1, sender 2 의 인코딩 된 결과가 받는쪽에 도착하지만, sender 1 의 code 를 가지고 있다면 받는 쪽은 온전히 sender 1 의 데이터를 해독할 수 있다. 


## 3. WiFi : 802.11 Wireless LANs

802.11 Wifi 에도 여러가지가 있다. 802.11a, 802.11b, 802.11g ... 그렇지만 이들 모두 같은 medium access protocol 인 CSMA/CA 를 사용하고, 같은 link-layer frame 을 사용한다. 그리고 모두 base-station 이 있고 ad-hoc network version 이다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fe4af79b-49ee-4a50-a770-ce1b44b70b13">

마켓은 새로운 기술을 원했다. 시장의 요구로는 다음이 있다. 
- Demand for throughput 
  
  전세계의 50-80% 데이터가 WiFi 를 통해 전송된다. 
  
- New usage models / features
- Technical capabilities

802.11ax 가 밀집된 환경을 위해 나왔다. 목표는 밀집된 환경에서 WLAN deployment 의 성능을 높이는 것이다. 이것는 공간적 (MU-MIMO) 와 주파주 멀티플렉싱 (OFDMA) 를 통해 이루어졌다. 

<img width="350" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/407bf5c3-0ce6-40b9-b041-d5907231a46f">

### 802.11 Architecture

802.11wireless LAN 의 구조를 보자. 가장 기본적인 것은 **<span style="background:#FEFBD1">basic service set (BSS)</span>** 이다. BSS 는 1개 이상의 wireless station 과 중앙의 **<span style="background:#FEFBD1">base station(=access point, AP)</span>** 이 있다. 이더넷과 마찬가지로, 802.11 wireless station 은 NIC에 6-byte MAC 주소가 있다. 각 AP는 wireless interface 에 MAC 주소가 있다. 


<img width="300" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7cd6701f-9a51-4fbf-9426-d03567e86717">

만약 중앙 제어가 없고, 외부 세상에 연결이 없으면 station 끼리 모여서 ad-hoc network 를 구성할 수 도 있다. 

#### Channels and Association

802.11 에서는, 각 wireless station 는 네트워크 레이어 데이터를 받기 전에 AP 와 연관(associate)해야 한다. 네트워크 관리자가 AP 를 설치하면, 관리자는 1개 혹은 2 단어의 **<span style="background:#FEFBD1">Service Set Identifier(SSID)</span>** 를 AP 에 할당한다. 그리고 관리자는 AP 마다 **<span style="background:#FEFBD1">channel number</span>** 를 할당해야 한다. 

802.11 은 2.4 GHz - 2.4835 GHz 주파수 범위 내에서 작동한다. 이 85MHz band 내에서, 802.11 는 11개의 부분적으로 겹치는 채널을 정의한다. 두 채널이 4개 이상의 채널로 인해 분리되면 두 채널은 절대 겹치지 않는다. 특별히, 채널 1,6,11 은 유일하게 겹치지 않는 3개의 채널이다. 

이것은 네트워크 관리자가 3개의 802.11b 를  물리적으로 같은 곳에 설치하면,
최대 33Mbps 의 전송률을 가진 wireless LAN 을 만들 수 있다는 뜻이다. 

<img width="500" alt="스크린샷 2023-12-10 오후 8 34 20" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a7e3fa9a-bf35-492e-9e74-21570b5e5721">

**<span style="background:#FEFBD1">WiFi jungle</span>** 이란 2개 이상의 AP 에게서 충분히 강한 신호를 받는 물리적인 위치를 뜻한다. 예를 들어, 카페에 앉아 있을 때 2개 이상의 와이파이 신호가 잡힐 수 있다. WiFi jungle 에 5개의 AP 가 있다고 하자. 인터넷에 접속하려면, 내 디바이스는 정확히 1개의 AP에 연결해야 한다. 연결한다는 것은 내 무선 디바이스가 AP 와 가상의 wire 를 만드는 것이다. 연결된 AP 는 data frame 를 내 무선 디바이스로 보낼 것이다. 그리고 내 무선 디바이스는 AP 를 통해서 인터넷에 data frame 을 보낼 것이다. 하지만 여기서 내 디바이스는, WiFi jungle 에 어떤 AP 들이 있는지 어떻게 알까?

802.11 표준은 AP 가 주기적으로 **<span style="background:#FEFBD1">beacon frame</span>** 을 전송하는 것을 요구한다. beacon frame 에는 AP의 SSID 와 MAC 주소가 적혀 있다. 무선 디바이스는 AP 가 beacon frame 을 전송하는 것을 알고 있고, beacon frame 을 보내는 AP를 찾는다. 

이때 찾는 방식에는  **<span style="background:#FEFBD1">passive scanning</span>** 과 **<span style="background:#FEFBD1">active scanning</span>** 이 있다. 
- **passive scanning**
	- (1) beacon frames sent from APs
	- (2) association request frame sent : H1 to selected AP
	- (3) association Response frame sent from selected AP to H1
    <img width="300" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e9581cda-cb63-43bd-ab1b-807c3dc7307c">
- **active scanning** 
	- (1) Probe request frame broadcast from H1
	- (2) Probe Response frames sent from APs
	- (3) Association Request frame sent : H1 to selected AP
	- (4) Association Response frame sent from selected AP to H1
  <img width="300" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/90e5534e-ddb7-4104-8ef7-1666e109b08d">

#### Ad Hoc Networking 

Ad Hoc networking 에서는 AP 가 없고, "peer-to-peer" 모드이다. 무선 호스트들은 서로와 소통한다. 무선 호스트 A에서 B로  패킷을 전달하려면, 무선 호스트 X,Y,Z 를 통과해야 될 수 도 있다. 

사용 예시는 회의장에서 노트북 미팅을 진행하거나, 개인 디바이스끼리의 연결, 전쟁에서 소통할 때 이다. 

### 802.11 MAC Protocol : CSMA/CA

무선 디바이스가 AP 와 연결되면, AP 에게 데이터를 전송할 수 있고 전송받을 수 있다. 다수의 무선 디바이스가 있기 때문에, AP 는 동시에 같은 채널로 data frame 을 보내고 싶을 수 있다. 따라서 이 전송을 제어하기 위한 **multiple access protocol** 이 필요하다. 앞에서, 3가지 종류의 multiple access protocol : channel partitioning, random access, taking turns 가 있다고 했다. 802.11 의 설계자들은 random access protocol 을 골랐다. 이것을 **<span style="background:#FEFBD1">CSMA with collision avoidance (CSMA/CA)</span>** 라고 한다. 

이더넷의 CSMA 와 같이, 여기서도 CSMA 는 "carrier sense multiple access" 이다. 즉, 각 스테이션 (AP) 은 전송하기 전에 채널을 탐지한다. 그리고 채널이 바쁘면 전송하지 않는다. 

2가지가 비슷해 보여도 중요한 차이점이 있다.  첫번째로, 802.11 은 collision detection(CD) 을 사용하는 대신, **<span style="background:#FEFBD1">collision avoidance(CA)</span>** 를 사용한다. 두번째로, 무선 채널에 bit error 가 많기 때문에, 802.11 은 **<span style="background:#FEFBD1">link-layer acknoledgment / retransmission scheme (ARQ)</span>** 을 사용한다. 

이더넷의 collision detection 에서는, 이더넷 스테이션은 채널에 전송하기 전에 채널을 탐지한다. 만약 전송 중에 다른 스테이션이 전송한다면, 전송을 중지하고, 랜덤한 시간동안 기다린 다음 다시 전송한다. 802.3 이더넷 프로토콜과는 다르게 802.11 MAC 프로토콜은 collision detection 을 하지 않는다. 이렇게 하는 2가지 이유는 다음과 같다.

1. collision detection 을 하는데는 시그널을 보내는 능력과 , 다른 스테이션이 전송 중인지 알아내는 능력 2가지가 동시에 필요하다. 이더넷에 비교해서 받는 시그널의 세기가 현저히 작기 때문에, collision detection 을 할 수 있는 하드웨어를 만들기는 비용이 많이 든다. 
2. adapter 가 전송하고 탐지하는 것을 동시에 할 수 있다고 해도, hidden terminal problem 때문에 모든 충돌을 감지할 수 없을 것이다. 

802.11 은 collision detection 을 하지 않기 때문에, frame 을 전송하기 시작하면 끝까지 다 보낸다. 하지만 이 방식은 충돌이 일어날 경우 프로토콜의 성능을 현저하게 저하시킨다. 그래서 충돌을 줄이기 위한 방식들을 한다. 

collision avoidance 를 알아보기 전에, **<span style="background:#FEFBD1">link-layer acknowledgment</span>** scheme 에 대해 먼저 알아보자. wireless LAN 에 있는 스테이션이 frame 을 전송하면, frame 은 다양한 이유로 목적지에 도달하지 않을 수 있다. 따라서 이 실패를 만회하기 위해, 802.11 은 link-layer acknowledgment 를 사용한다. 만약 목적지 스테이션이 CRC 체크를 통과하는 frame 을 받으면, **<span style="background:#FEFBD1">Short Inter-frame Spacing(SIFS)</span>** 를 기다렸다가 ACK 를 보낸다. 만약 전송하는 스테이션이 일정 기간 내에 ACK 를 받지 못하면, 에러가 발생했다고 생각하고 frame 을 재전송한다. 만약 고정된 수의 재전송 후에도 ACK가 오지 않으면, 스테이션은 전송을 포기하고 frame 을 버린다. 

<img width="257" alt="스크린샷 2023-12-10 오후 9 47 25" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/afeb5522-0f6d-4af2-86ac-9fdfd73a8d8e">

이제 CSMA/CA 프로토콜이 어떻게 작동하는지 보자. 스테이션 (wireless device or AP) 가 전송할 frame 이 있다고 하자. 

1. 스테이션이 채널이 idle 하다고 탐지하면, **Distributed Inter-frame Space(DIFS)** 만큼 기다렸다가 frame 을 전송한다. 
2. 채널이 바쁘다고 탐지하면, 채널은 binary exponential backoff 를 사용해서 랜덤한 backoff 값을 선택한다. DIFS 이후 채널이 idle 하다고 탐지하면 DIFS 을 카운트다운 한다.  만약 채널이 바빠지면 카운터를 멈춘다.
3. 카운터가 0에 도달하면, 스테이션은 frame 전체를 전송하고 ACK 를 기다린다. 
4. ACK 가 도달하면, 스테이션은 frame이 성공적으로 전달된 것을 안다.
5. 만약 ACK 가 도착하지 않으면, random backoff interval 을 증가시키고 2번부터 다시 한다. 

**받는 쪽**의 입장을 보자.

Frame 을 성공적으로 전달 받으면, SIFS 이후 ACK 를 전송한다. 

-----

