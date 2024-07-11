+++
author = "Soeun"
title = "코딩 일지: Daily Coding Times 제작 후기"
date = "2024-07-11"
summary = "일간 코딩 신문 배달왔습니다 !"
categories = [
    "Project"
]
tags = [
    "daily coding times"
]
+++

## 소개

오랜만에 아주 재미있는 프로젝트를 했습니다. 어렸을 때 해리포터 영화를 보면서, "The Daily Prophet" 을 보고 굉장히 신기해했던 기억이 있습니다. (이 당시에는 아이패드는 커녕 2G폰을 쓰고 있을 때였죠...) 어렸을 때의 추억도 되살릴겸, 저를 소개하는 신문을 만들어보았습니다. 

여기에 제가 추가하고 싶던 기능은, 지난 일주일 간 제가 했던 커밋 내용을 불러와서, 이것을 요약해주고 응원해주는 메세지를 담고 싶었습니다. (험난한 개발시장에서 잘하고 있다는 응원을 받고 싶은 저만의 자존감 지킴이랄까...)  그리고 제가 자주 확인하는 이번주 GitHub Trending 레포들을 소개하고 싶었습니다. 더불어 신문이니 오늘의 날씨도 알려주고자 했습니다. 


<img width="1616" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4be04fb8-a832-444b-88e5-393ba4dce86a">

실제 배포한 웹사이트는 [여기](https://daily-coding-times.vercel.app/)서 확인할 수 있습니다. 아래는 제가 이 웹페이지를 개발하면서 마주친 고민들과 해결 과정을 소개해보겠습니다.

## 사용한 기술

- 프런트 : javascript, css, html
- 백엔드
    - 서버 : express
    - 데이터베이스 : Cloud firestore 
    - 인프라 : vercel

우선 저는 프런트엔드 개발 경험이 전무(?) 합니다. 따라서 리액트, Vue 등 프레임워크를 사용하지 않고 javascript, css, html 만 사용해서 구현했습니다. css 애니메이션을 조금 공부한게 크게 도움이 되었습니다. 특히 신문 레이아웃을 만들기 위해 css grid를 많이 사용했습니다. 

서버로는 **express** 를 사용했습니다. 프런트와 백엔드 개발을 하나의 언어로 할 수 있다는 장점이 있어서 선택했습니다. 데이터베이스로는 **firestore** 를 선택했는데, nosql database이어서 javascript 와 궁합이 잘 맞는다는 장점이 있습니다. 비용적인 측면에서도 AWS RDS 또는 Google Cloud Storage 를 사용하는 것보다 firestore가 저렴했습니다. 또 저는 firebase의 functions 를 사용해서 매일 자정마다 깃헙 커밋 요약을 업데이트하고자 했기 때문에, 자연스럽게 firebase 내에 있는 firestore 를 선택하게 되었습니다. 배포는 **vercel**을 사용했는데, 이번에 처음 사용해봤는데 너무 편리하고 간편해서 아주 만족스럽습니다. 로컬 cli 에서도 vercel 명령어로 dev 환경을 미리 볼 수도 있고, 바로 배포를 할 수 있습니다. 또 깃헙 레포지토리와 연동하면 깃헙에 푸쉬할 때 변경사항이 바로 배포되도록 할 수 있었습니다. 


## 매일 업데이트 되는 신문 : firebase의 functions 사용하기

### 깃헙 커밋 요약

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a2bb86d5-1965-4041-9e02-c5164b9a7a61 '매일 업데이트되는 커밋 요약')

최근 일주일 간의 커밋 내역을 가져와서, OpenAI API 를 사용해 요약과 응원의 메세지를 생성하게끔 구현했습니다. 이 함수를 script.js 에서 바로 사용하니 페이지를 로드할 때마다 OpenAI API 요청이 발생했습니다. 이를 막기 위해 매일 자정에 한번 요약을 생성하여 데이터베이스에 저장하고, 페이지를 로드할 때는 데이터베이스에서 읽어오도록 구현했습니다. 매일 자정에 요약이 업데이트되도록 하기 위해서 **firebase의 [cloud functions](https://firebase.google.com/docs/functions)** 를 사용했습니다. 

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b22be172-b621-4a48-b62f-de908333fc14 'Firebase Cloud Functions')

### 깃헙 트렌딩 레포 

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/793dfe55-8b17-4e52-93a4-d0cf88388f1a '일간 깃허브 트렌딩 레포')

깃허브에서 현재 트렌딩되고 있는, 스타의 수가 가장 많은 레포를 가져오기 위해 크롤링을 했습니다. 매일 자정에 크롤링해서 데이터베이스에 저장하도록 cloud functions 를 설정했습니다.

## 현재 날씨 보기

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d9e449bb-936a-4fd4-bfa3-db616280327f '현재 날씨')


현재 날씨를 가져오기 위해서는 [OpenWeather API](https://openweathermap.org/api) 를 사용했습니다. 날씨에 맞는 귀여운 이모지는 [Microsoft Fluent Imoji](https://animated-fluent-emoji.vercel.app/) 를 사용했습니다. 그 결과 아주 귀여운 날씨 화면이 완성되었습니다.

## 후기

매일 업데이트되는 부분 외에는, 제가 한 프로젝트 소개와 저에 대한 소개글을 넣었습니다. 그 결과 꽤나 신문같은 웹사이트가 완성되었습니다. 해리포터 신문을 실제로 웹 페이지로 구현해보는게 너무나도 재밌었고, 어떤 내용을 넣을지 고민하면서 채워가는 과정이 즐거웠습니다. 딱딱한 학교 공부에서 벗어나 오랜만에 재밌는 프로젝트였습니다. 디자인, 개발, 배포까지 해본 첫 프로젝트로 기억에 남을 것 같습니다. 코드는 아래 레포에서 볼 수 있습니다. 

{{< github repo="ddoddii/Daily-Coding-Times" >}}