+++
author = "Soeun"
title = "CJ대한통운 미래기술챌린지에서 느낀 점"
date = "2023-08-21"
description = "우리는 어떻게 비정제 영문 주소 번역 문제에 접근했는가"
categories = [
    "Project"
]
tags = [
    "CJ공모전"
]
draft = true
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ea9adcdd-3652-4c0e-a96a-737b66572aa1"
+++

## 과제 소개
활동했던 학회에서 팀원 3명과 함께 CJ대한통운 미래기술챌린지 2023에 참여했다. 우리 팀은 과제 4 - 비정제 영문 주소 번역 시스템에 참여했다.

주어진 과제는 **다양한 형식의 비정제 영문 주소를 활용하여 한국 주소 체계에 맞는 영문주소 AI번역 시스템 구현**하는 것이었다. 

평가 방식은 20,000건의 결과 반환 시간 측정 (40%) + 데이터 정확도 (60%) 이었다.

모델의 정확성, 범용성, 효율성 모두 고려해야 했다. 

## 데이터 소개

주어진 데이터는 영문 주소 데이터였다. 아래와 같이 오류가 있는 케이스들이 있었다.
|번호|오류케이스|예시|
|:---:|-------|---|
|01|영문주소 + 배송요구사항|Seongbu Tax Office, 13, Samseoyeon-ro 16-gil, Seongbuk-gu, Seoul경비실 맡겨주세요|
|02|영문주소 + 한글주소|229, 당산로, Yeongdeungpo-gu, Seoul, Republic of Korea|
|03|영문주소 + 특수기호|B1721, Nambu Ring-ro, Gwanak-gu, Seoul 김&장|
|04|표준 주소 입력 체계 무시|B102 Dosan-daero Seoul (Sinsa-dong)Republic of Korea|
|05|주소 오입력|B 2, Sejong-daero, Gung-gu, Seoul|

위와 같이 오류가 있는 영문 주소를 온전한 한글 주소로 바꾸는 과제였다. 한글 주소의 기준은 도로명 주소에서 검색 여부로 판단했다.  

## 접근 방식 
- 원본 데이터(B93, Wangsan-ro, Dongdaemun-gu, Seoul 1001ho 1001동) 를 정제된 한글 주소(서울특별시 동대문구 왕산로 지하 93)로 바꾸려면 어떻게 해야 할까?
- 주소기반산업지원서비스 API 에서 '왕산로 지하93'을 검색하면 완전한 한글 주소를 얻을 수 있다.
- 원본 데이터에서 번지(도로명 또는 지번) + 건물명 또는 상세주소 레벨의 주소를 추출해 번역한 후 위의 api 를 위용하면 해결할 수 있겠다고 생각했다.

### Ideation

### 사용한 모델- t5
t5 는 Text-to-Text Transfer Transformer 모델이다. t5는 모든 NLP task 를 text-to-text 형태로 전환시키는 통합된 프레임워크를 제시한다. 