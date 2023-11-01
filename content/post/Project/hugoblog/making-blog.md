+++
author = "Soeun"
title = "Hugo blog 기초부터 만들기"
date = "2023-11-01"
description = "나의 커스텀 블로그 구축기"
categories = [
    "Tips"
]
tags = [
    "hugo blog"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d3896a1f-8041-49d3-beab-02e1897090be"
slug = "making-hugo-blog-from-scratch"
+++

나는 작년부터 깃블로그를 만들기 위해 jekyll, hugo 등 여러 방법을 시도해보았다. 나의 블로그 선택 기준은 아래과 같다.

1. 사용하기 쉬움 
2. 빠른 deploy
3. 빠른 로딩 속도
4. 내가 커스텀 가능한 디자인 

jekyll은 ruby 를 설치해야 했고, delpoy하는 속도가 느리고 내가 마음에 드는 테마도 없었다. m1 mac 을 사용하는 나에게 ruby 를 사용하는것이 가장 큰 장벽이었다.. 그래서 찾고 찾다가, 정말 마음에 쏙 드는 블로그를 발견했는데, hugo 로 만들어졌다는 것을 발견했다. 그 후로 [hugo 공식 사이트](https://gohugo.io/) 를 보면서 하나하나 직접 만들었다. 

블로그 기능 중에서도 내가 꼭 원하는 기능들이 있었다. 

1. 카테고리 
2. 태그
3. 목차(table-of-contents)
4. 날짜, 읽는 시간 표시
5. Light/dark mode 지원

나의 이 까다로운 조건들을 만족하기 위해 hugo 내에 있는 모든 테마들을 보았고, 드디어 내가 200% 만족하는 테마에 정착해서 잘 커스텀해서 사용하고 있다 ! 현재 이 블로그가 그 결과물이다. 

그럼 지금부터, 내가 0부터 시작해서 어떻게 블로그를 만들었고, 어떻게 관리하고 있는지에 대해 설명해보겠다. 

## 첫 시작

우선, 공식 문서는 여기있다. [hugo quick start](https://gohugo.io/getting-started/quick-start/) 이 문서만 보고 따라해도 쉽게 만들 수 있지만, 영어로 되어 있어서 다시 차근차근 설명해보겠다.

### Prerequsite
- Install hugo
	- mac : `brew install hugo`
	- window : [window install hugo](https://gohugo.io/installation/windows/)
- Install git (기본이라서 넘어가겠다.)

### Create a site
내가 맥을 쓰므로, 맥 기준으로 설명하겠다. 터미널에 아래 명령어들을 입력하면 된다. 
아래에서는 좀 예뻐보이고 내 스타일인 [hugo-tranquilpeak-theme](https://github.com/kakawait/hugo-tranquilpeak-theme) 테마를 사용했다. 더 많은 테마들은 [여기](https://themes.gohugo.io/) 서 확인할 수 있다. 다른 테마를 적용하고 싶다면, git submodule add "내가 원하는 테마의 깃 저장소" themes/"테마 이름" 이렇게 변경하면 된다. 보통 테마마다 설명서가 다 있다. 

```bash
hugo new site quickstart #quickstart : 내가 블로그를 담고 싶은 폴더 이름 (나의 경우에는 my-blog 로 했다.)
cd quickstart  #위에서 만든 디렉토리로 이동
git init 
git submodule add https://github.com/kakawait/hugo-tranquilpeak-theme.git themes/hugo-tranquilpeak-theme
echo "theme = 'hugo-tranquilpeak-theme'" >> hugo.toml 
hugo server
```

진짜 쉽다. 여기까지 하면 로컬에서 내 블로그를 확인하는 localhost 주소가 나타날 것이다. 
<img width="700" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8fe864fa-5161-45e2-8ef4-c5485ae80a4e">

http://localhost:1313/  에 들어가보면 .. 두둥! 

<img width="700" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0225c32c-36ef-41c5-acdd-c9edb64c68c4">

아무것도 안나온다😂 이게 정상적이다. 왜냐하면 아직 아무런 Content 를 안 넣었으니까 ! 차근차근 채워나가보자. 
### themes/exampleSite
themes/hugo-tranquilpeak-theme 아래 디렉토리에 가면, exampleSite 라는 폴더가 있다. 이거를 잘 봐야한다  !! 이게 데모 사이트가 이루어진 사이트이다. 이것을 참고하면서, 나의 블로그를 만들어가면 된다. 

<img width="308" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3bcbfac2-41ad-459c-b10a-c227e25beef8">

디렉토르가 이렇게 생겼을 것이다. 그러면 우리의 root 디렉토리에도 content 폴더가 있을 것이다. 여기가 바로 우리의 글들이 들어갈 폴더이다. 똑같이 content/posts 폴더를 만들어주고, 첫번째 글을 써보자. 

글의 형식도 중요하다. exampleSite 내에 .md 아무거나 클릭해보면, 

<img width="700" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/88e48b46-407e-4345-8bab-a191d6890353">
위와 같은 메타데이터가 있을 것이다. 이것까지 똑같이 복사해서, 첫번째 포스트를 발행해보자.

<img width="700" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e6b291a1-03bb-4f9f-88aa-67cb5e2458ae">
그러고 다시 hugo server 를 입력해서 로컬호스트에 들어가보면 !! 

<img width="700" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/344ce39d-2ff0-4fb8-9ae7-c60649806f04">

오 뭔가 생겼다 ! 근데 클릭할 수 없다 .. 왜냐면 content 만 넣어줬지, 사이트 구조는 여전히 비어있는 깡통이기 때문이다. Hugo blog 를 세팅하는데 가장 중요한 것은 `config.toml` 이다. 이것을 잘 만져야 한다. 좀 쉽게 하기 위해서, exampleSite 에 있는 config.toml 을 복사해서, 내 config.toml 에 붙여 넣자. 그 후 hugo server 를 통해 , 다시 로컬호스트에 접속해보자. 

<img width="700" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/abd9da3b-5a65-4d1c-b7ed-52771a6c8bf4">

축하한다 !! 👏 드디어 hugo blog 에 대한 첫번째 세팅을 마쳤다 ! 

config.toml 을 보면, 주석 처리된 설명이 굉장히 많다. 여기서 내가 커스텀 하려면 어떤 것을 바꿔야 할지, 구글링 해보면서 직접 알아가보자. 어렵지 않다 ! 

`hugo server -D` 를 사용하면, 로컬호스트를 껐다가 킬 필요 없이 변경사항이 새로고침하면 바로 반영된다. 

## 글 쓰기 

블로그에서 가장 중요한 단계이다. 바로 글 쓰기 ! 기본적으로 마크다운 형식으로 작성한다. 

### 디렉토리 구조

<img width="285" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/56326309-0514-4481-8817-faec981ab988">

기본적으로 content/posts 아래 글을 작성하면 된다. 그 외 폴더 layouts, public, resources, static 에 대해서는 차차 설명하겠다 .. 

커스텀 하는 것은 exampleSite, 다른 hugo blog 의 디렉토리 구조를 살펴보고 혼자 실험해보길 바란다. 이게 제일 빠른 방법이다. 

## 배포 (with github pages)

지금까지는 내 블로그를 로컬호스트를 통해서만 봤다. 이제 github pages 로 실제 작동하는 url 을 만들어보자. 물론 firebase, netlify 등 다른 호스팅 서비스도 있지만 우리에게는 github 가 가장 익숙하다. [Host on github pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

### [githubid].github.io 만들기

github 에 들어가서, [본인깃헙아이디].github.io 이름의 레포를 하나 만들어라. github 는 사용자마다 1개의 정적 페이지를 호스팅할 수 있게 해준다. 

Settings > Page 의 Build and deployment 에서 Source 를 GitHub Actions 로 바꾸자.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/149598069/5b898473-834e-437f-b97a-eb1d60d3bf18)

그 후 다시 vscode 로 와서, 루트 디렉토리에 .github/workflows 디렉토리를 만든 후, 아래 hugo.yaml 을 복사해서 넣는다.

```yaml
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.115.4
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb          
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"          
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2>)
```

그 후 , 본인 로컬 레포와 깃헙 레포를 remote add origin 을 통해 연결해준 후, git add, commit 하고 Push 해준다. 저 Workflow 덕분에 내가 변경 사항을 업로드 할 때마다 알아서 자동으로 배포해준다. 

여기까지 Hugo blog 구축하기 기초이고, 다음 글에서는 blog 를 구글 search console 에 등록하는 방법, google analytics 로 방문자 수 추적하는 방법까지 알아보겠다. 

