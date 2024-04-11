+++
author = "Soeun"
title = "Github Actions 캐시 기능 사용하여 자동 배포 시간 단축하기"
date = "2024-02-07"
summary = "Github Actions '잘' 활용하는 방법"
categories = [
    "CS"
]
tags = [
    "Github Actions",
    "CI/CD"
]
slug = "github-actions"
+++

프로젝트에서 자동 배포 파이프라인을 구축하기 위해, **Github Actions** 를 사용했습니다. 여러 CI/CD 파이프라인이 있지만, 깃헙 액션을 선택한 이유는 아래와 같습니다. 

1. 쉬운 yaml 문법으로 인해 러닝 커브가 낮았습니다.
2. 깃허브 베이스여서, 사용성이 뛰어났습니다. 깃 레포지토리에 푸쉬하면 자동으로 배포되게 하는 등 사용이 직관적으로 쉬웠습니다.

프라이빗 레포에서는 깃헙 액션을 사용하는 시간마다 비용이 청구되기 때문에, 이 시간을 줄이는 것이 중요합니다. 따라서 저도 단순히 자동화하는 것을 넘어서, 캐시 기능을 사용하여 깃헙 액션 시간을 단축했습니다. 캐시 기능 사용 전에는 3분 정도 걸리던 깃헙 액션 시간이, 2분 내외로 단축되었습니다. 

본 포스팅에서는 Github actions 이 어떻게 동작하고, '잘' 활용하려면 어떻게 해야 하는지 작성해보았습니다. 

## Github Actions ?

Github Actions 는 **CI/CD 플랫폼**으로, 빌드, 테스트, 배포 파이프라인을 자동화할 수 있습니다. 풀 리퀘스트, 머지, 새로운 이슈 등 사용자가 여러 **이벤트**를 설정하고, 그 이벤트가 일어났을 때 자동으로 동작하는 **워크플로우**를 만들 수 있습니다. 워크플로우는 .github 폴더 내에 `workflows.yaml` 파일을 작성해서 정의할 수 있습니다.

깃헙은 워크플로우를 실행하기 위한 리눅스, 윈도우, 맥OS등  다양한 **가상 머신**을 러너(runner)로 제공해줍니다. 또한, 데이터 센터나 클라우드 인프라를 러너로 등록해 사용할 수 도 있습니다. 

레포지토리에 *이벤트(Event)* 가 발생하면, Github Actions *워크플로우(Workflow)* 가 시작되도록 설정할 수 있습니다. 워크플로우는 순차적으로 실행되거나 병렬적으로 실행되는 여러 *잡(Job)* 들의 모음입니다. 각 잡은 각자 가상 머신 *러너(runner)* 혹은 컨테이너 내에서 실행됩니다. 잡은 액션 내에서 정의한 스크립트를 실행하거나 재사용가능한 익스텐션을 실행하는 하나 이상의 스텝으로 이루어져 있습니다.  

<img width="520" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ced04ff6-7f81-47ef-8792-d649fe57500f">

### Workflows

워크플로우는 자동화된 프로세스로, 1개 이상의 job 들을 실행합니다. 워크플로우는 깃헙 레포 내에 `.github/workflows` 폴더 내 yaml 파일을 작성해서 정의할 수 있습니다. 워크플로우의 예시로는 풀리퀘스트를 빌드하고 테스트하는 것, 릴리즈가 출시되었을 때 자동으로 배포하는 것, 다른 사용자가 새로운 이슈를 열었을 때마다 레이블을 추가하는 것 등이 있습니다. 

### Events

이벤트는 내가 정의한 워크플로우가 트리거 되는 상황입니다. 예를 들어서, 릴리즈를 자동 배포하는 워크플로우는 내가 메인 브랜치로 머지했을 때 일어나게 하고 싶다면 "메인 브랜치로 머지" 가 바로 이벤트가 되는 것입니다. 

### Jobs

Job는 워크플로우 내에 있는 세부적인 스텝들의 모음입니다. Job 은 같은 러너에 의해 실행됩니다. 각 스텝들은 순차적으로 실행되고, 서로에게 의존성이 있습니다. 각 스텝이 같은 러너에 의해 실행되기 때문에, 스텝끼리는 데이터를 공유할 수 있습니다. 

반면 Job 간에는 기본적으로 아무런 연관관계가 없고, 병렬적으로 실행됩니다. Job 간에 연관관계를 설정해줄 수 도 있습니다. 그러면 이전의 job 이 끝난 후에 다음 job 이 실행되게 됩니다. 

### Actions

액션은 깃헙 액션 플랫폼에 있는 커스텀 어플리케이션으로, 자주 반복되는 태스크들을 정의해 놓은 것입니다. 깃헙 액션들의 도움을 받아서 내 워크플로우 내에 있는 반복되는 코드들을 줄일 수 있습니다. (마치 라이브러리와 같은 개념입니다.)

### Runners

러너는 워크플로우가 트리거되었을 때, 워크플로우를 실행하는 서버입니다. 각 러너는 하나의 job 을 실행할 수 있습니다. 깃허브는 리눅스, 윈도우, 맥OS 등 다양한 러너를 제공합니다. 각 워크플로우는 매번 초기화된 상태의 가상머신에서 실행됩니다. 

위의 설명에서 알 수 있듯이, 매 워크플로우는 실행할 때마다 초기화된 가상머신에서 실행됩니다. 하지만 제 워크플로우 내에는, pip 모듈을 다운 받는 것과 같이 자주 바뀌지 않지만 실행하는데 오래 걸리는 스텝들이 포함되어 있었습니다. 

깃헙 액션에서도 이러한 워크플로우를 위해 **dependency들을 캐싱하는 기능**을 제공합니다. 이 기능에 대해 자세히 알아보겠습니다. 

## Caching dependencies to speed up workflows

깃허브에서 호스팅하는 러너들을 사용하면, 매번 새로운 러너로 시작합니다. 하지만 매번 dependencies 를 다운 받는 것은 네트워크 사용량을 증가시키고, 실행시간이 길어지고, 비용이 더 많이 듭니다. 따라서 깃헙는 자주 사용되는 파일을 캐싱할 수 있게 해줍니다. 

캐싱 기능을 위해서는 깃허브의 [cache action](https://github.com/actions/cache) 을 사용하면 됩니다. 이 액션은 유니크한 키에 의해 캐시를 생성하고, 가져옵니다. 

### `cache` action 사용하기

캐시 액션은 사용자가 제공한 key 를 가지고 캐시를 가져옵니다. 이 key 는 리스트 형태(restore-keys)로도 제공할 수 있습니다.  액션이 key 와 정확히 매치하는 캐시를 찾으면, 사용자가 설정한 path 로 캐시를 복원합니다. 이것을 **cache hit** 라고 합니다. 만약 key 와 일치하는 캐시가 없다면, 이것은 **cache miss** 라고 합니다. 캐시 미스시에는, job 이 성공적으로 끝날 경우 액션은 자동으로 새로운 캐시를 생성합니다. 

이미 존재하는 캐시의 내용을 바꿀 수는 없습니다. 대신, 새로운 키를 가지고 새로운 캐시를 생성할 수는 있습니다. 

#### `cache` action 을 위한 인풋 파라미터들

- `key` : (필수) 캐시를 저장할 때 키가 생성되고, 캐시를 가져올 때도 키가 사용됩니다. 
- `path` : (필수) 러너가 캐시를 저장하거나 가져오는 경로입니다. 
  - 하나 또는 여러 개의 경로를 설정할 수 있습니다. 
  - 절대적 경로 또는 워크스페이스 디렉토리에서 상대적인 경로를 지정할 수 있습니다. 

    ```yaml
    - name: Cache Gradle packages
    uses: actions/cache@v3
    with:
        path: |
        ~/.gradle/caches
        ~/.gradle/wrapper
    ```

- `restore-keys` : (선택) 대체로 사용할 수 있는 키들을 나타냅니다. key 에 대해 캐시 미스가 발생하면, restore-keys 를 순차적으로 사용하여 캐시를 검색합니다. 

    ```yaml
    restore-keys: |
    npm-feature-${{ hashFiles('package-lock.json') }}
    npm-feature-
    npm-
    ```

- `enableCrossOsArchive` : (선택) boolean 값으로, true일 때는 윈도우 러너가 캐시가 생성된 OS에 상관없이 캐시를 저장하거나 가져올 수 있도록 합니다. 기본값은 false 입니다. 

#### `cache` action 을 위한 아웃풋 파라미터들

- `cache-hit` : 캐시 히트가 발생했는지를 나타내는 boolean 값입니다. 

## 실제 사용기

```yaml
name: ci

on:
  pull_request:
    branches:
      - "main"

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v4
      -
        name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      -
        name : Cache pip modules
        id : cache-pip
        uses : actions/cache@v3
        with :
          path : ~/.cache/pip
          key : ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-
            ${{ runner.os }}-

        // 기타 스텝 
```

저는 pip 모듈을 다운 받을 때, 캐시 기능을 사용했습니다. path는 `~/.cache/pip` 로 지정했으며 key 는 컨텍스트를 고려하여 지정했습니다. 

아래와 같이 액션을 실행할 때, 캐시 히트가 발생하여 Install dependencies 가 생략된 것을 볼 수 있습니다.

<img width="889" alt="스크린샷 2024-04-11 오후 3 38 32" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/23795f0c-05ee-41ed-9fc0-6896eb5e3f1c">


<img width="892" alt="스크린샷 2024-04-11 오후 3 38 21" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6d05bd1d-12ba-4b60-9aaf-7d279e3b097f">

## 추가적으로 생각해볼 점

깃헙 액션의 캐시 기능으로 모든 문제가 해결될까요? 그것은 아닙니다. 저는 프로덕션 환경이 아니라서 우선 여기까지 진행했지만, 다른 기술 블로그에서 깃헙 액션 기능을 사용한 사례를 찾아보았습니다. 

- [뱅크샐러드 Web chapter에서 GitHub Action 기반의 CI 속도를 개선한 방법](https://blog.banksalad.com/tech/github-action-npm-cache/)

위의 글에서도 깃헙 액션의 캐시 기능을 사용했습니다. 하지만 한계점도 같이 소개하고 있습니다. 

> 1) GitHub Action은 각 브랜치의 job마다 새로운 가상머신(runs-on에 명시된 운영체제)이 생성됩니다. 따라서 생성된 직후인 첫번째 commit에는 cache가 없으며 이후 두번째 commit 부터 cache가 존재하여 이를 활용할 수 있습니다.
> 2) runner의 최대 저장공간과 저장 기간은 plan에 따라 제한이 있습니다(e.g. Free plan: 10GB). 따라서 생성된 cache가 저장공간의 크기를 넘어서게 되면 오래된 cache부터 순차적으로 자동 삭제됩니다.
> 3) workflow의 event type이 deployment 일 때는 cache를 사용할 수 없습니다.

추가적인 해결책으로는 아래를 제시했습니다. 

1. Job 분리 

위에서 깃헙 액션을 소개할 때, Job 는 서로 연관관계가 없는 병렬적인 작업이라고 했습니다. 따라서, 실행순서가 상관없는 job 들을 여러 개로 나누어 병렬적으로 실행하면 더욱 효과적이라고 소개했습니다. 

2.  변경한 사항만 테스트하기 

git diff 를 사용하여 변경사항만 대상으로 한다면, 더욱 효과적으로 깃헙 액션 실행 시간을 단축할 수 있을 것입니다. 

이 점 역시 생각지도 못한 해결책인데, 추후에 적용 해봐야겠습니다. 🧐

- [모두의 Github Actions (feat. Github Enterprise) 3편 - Build Cache](https://hyperconnect.github.io/2021/12/21/github-actions-for-everyone-3.html)

하이퍼커넥트에 기술블로그에서도 비슷한 글이 있었습니다. 하이퍼커넥트 팀에서는 깃허브에서 제공하는 러너가 아니라 직접 호스팅하는 러너를 사용하고 있었습니다. 이때 빌드 캐시가 지원이 되지 않아서 직접 빌드 캐시를 만들었습니다.😮



## Reference
- [Understanding GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)
- [Caching dependencies to speed up workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#matching-a-cache-key)
- https://blog.banksalad.com/tech/github-action-npm-cache/
- https://hyperconnect.github.io/2021/12/21/github-actions-for-everyone-3.html