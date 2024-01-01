+++
author = "Soeun"
title = "Docker 에서 데이터를 관리하는 방법"
date = "2024-01-02"
summary = "Docker Volume , Bind Mount, 환경 변수 설정"
categories = [
    "CS"
]
tags = [
    "Docker"
]
slug = "docker-data-and-volumes"
series = ["Docker"]
series_order = 2
+++

## Data

웹사이트에서 발생하는 데이터는 여러 개의 종류가 있다. 

| 종류                                             | 특징                                                                                                                                                                                    |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Application<br>(Code + Environment)              | - 개발자에 의해 작성되고 제공된다.<br>- build 단계에 이미지와 컨테이너에 추가된다.<br>- "Fixed" 이다. 이미지가 빌드 된 이후에는 바뀌지 않는다. <br>- Read Only 이며, 이미지에 저장된다. |
| Temporary App Data<br>(e.g. entered  user input) | - 실행되고 있는 컨테이너에서 fetched / produced 된다. <br>- 메모리에 또는 임시 파일에 저장된다. <br>- Dynamic , changing <br>- Read + write, temporary, 컨테이너에 저장된다.            |
| Permanent App Data<br>(e.g user accounts)                                                 | - 실행되고 있는 컨테이너에서 fetched / produced 된다. <br>- DB 에 파일을 저장해야 한다. <br>- 컨테이너가 중지/시작되어도 계속 보존해야 한다. <br>- Read + write, permanent, container & volume 에 저장된다.                                                                                                                                                                                         |

![Untitled](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b0ac40da-b999-4d9e-aa6e-8a822527f5e0)


컨테이너에 저장된다는 것은 도커의 container layer 에 저장된다는 의미이다. 이 데이터는 이미지에도 없고, 로컬 머신에도 없고, 오로지 이 container layer 에만 있다. 

간단한 프로그램을 같이 보자. 

```javascript
const fs = require('fs').promises;  
const exists = require('fs').exists;  
const path = require('path');  
  
const express = require('express');  
const bodyParser = require('body-parser');  
  
const app = express();  
  
app.use(bodyParser.urlencoded({ extended: false }));  
  
app.use(express.static('public'));  
app.use('/feedback', express.static('feedback'));  
  
app.get('/', (req, res) => {  
  const filePath = path.join(__dirname, 'pages', 'feedback.html');  
  res.sendFile(filePath);  
});  
  
app.get('/exists', (req, res) => {  
  const filePath = path.join(__dirname, 'pages', 'exists.html');  
  res.sendFile(filePath);  
});  
  
app.post('/create', async (req, res) => {  
  const title = req.body.title;  
  const content = req.body.text;  
  
  const adjTitle = title.toLowerCase();  
  
  const tempFilePath = path.join(__dirname, 'temp', adjTitle + '.txt');  
  const finalFilePath = path.join(__dirname, 'feedback', adjTitle + '.txt');  
  
  await fs.writeFile(tempFilePath, content);  
  exists(finalFilePath, async (exists) => {  
    if (exists) {  
      res.redirect('/exists');  
    } else {  
      await fs.rename(tempFilePath, finalFilePath);  
      res.redirect('/');  
    }  
  });  
});  
  
app.listen(80);
```

사용자의 피드백을 받는 간단한 node 어플리케이션이다. 터미널에서 `node server.js` 를 실행하고 localhost를 방문하면, 아래와 같은 창이 뜬다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ff7e2dee-363a-442d-bb3d-c2756b348285)


여기서 title, document text 를 입력하면, 로컬 머신의 feedback 폴더 안에 저장된다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7b7a5bfe-7484-4298-a4eb-9e6aad0ac34b">

이제 Dockerfile 을 만들어서 Dockerize 해보자. 

```Dockerfile
FROM node:14  
  
WORKDIR /app  
  
COPY package.json .  
  
RUN npm install  
  
COPY . .  
  
EXPOSE 80  
  
CMD ["node", "server.js"]
```

`docker build -t feedback-node .` : 이미지 빌드 

`docker run -p 3000:80 -d --name feedback-app --rm feedback-node ` : 컨테이너 실행. 위의 어플리케이션에서 app.listen(80) 이었으므로 80포트를 3000포트로 매핑해주었다. `--rm` 은 컨테이너를 중지시키면, 컨테이너를 삭제한다. 

[http://localhost:3000/](http://localhost:3000/) 접속하면, 똑같은 창이 뜬다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9fe4874f-5511-4d86-a869-a4b705ca9213)

여기서 똑같이 Title, Document text 를 입력해보자. 저장된 파일은 도커 컨테이너 내부에서만 접속 가능하다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/163f6063-5f0f-45f1-8ec3-6206ed7a2108 "feedback 입력")

http://localhost:3000/feedback/good.txt 에 접속하면 내용을 확인 가능하다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7e216880-3cf5-462b-9789-00f5c791f73d)

하지만 **내 로컬 머신의 feedback 폴더 안에 저장되지 않는다** ! 왜냐면, 이 파일은 도커 컨테이너 내부에만 존재하기 때문이다. 도커 파일을 다시 보면, 우리는 `COPY . . ` 을 통해서 로컬에 있는 파일들을 이미지에 복사했다. 이때 이미지에는 로컬 폴더에 대한 **스냅샷**이 생기고 격리된 파일 시스템을 갖게 된다. 복사된 이후에는, 더 이상 우리의 로컬 폴더와 이미지 내부의 파일 시스템 사이에 연결은 없다. 컨테이너는 이 이미지를 기반으로 실행된다.  원래 도커가 이렇게 설계된 것이다. 왜냐면 컨테이너는 로컬 파일 시스템과 격리되어야 하기 때문이다 !

비슷한 원리로, 로컬에서 소스 코드를 변경해도, 컨테이너에는 아무런 영향이 없다. 변경된 코드를 반영하기 위해서는 이미지를 다시 빌드하고 컨테이너를 실행해야 한다. 

가장 중요한 점은, **컨테이너/ 이미지와 로컬 파일 시스템 간에는 연결 관계가 없다**는 것이다 ! 

## Problem - Data inside Container is lost

여기서 문제 상황이 발생한다. 우선 컨테이너를 중지해보자. (`docker stop feedback-app`) 그리고 다시 실행해보자 (`docker run -p 3000:80 -d --name feedback-app --rm feedback-node `) 

http://localhost:3000/feedback/good.txt 에 접속하면 오류(Cannot get /feedback/good.txt) 가 뜨는 것을 볼 수 있다.  

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c8e26817-2fc3-4822-9e2d-281220be0af2)

즉, 컨테이너를 **삭제(remove)** 했다가 다시 실행하면, 컨테이너 레이어에 저장된 데이터는 사라진다.  그렇다면, 컨테이너를 삭제하지 말고 **중지**했다가 재실행해도 데이터가 삭제될까? 아니다. 단순히 컨테이너를 중지하고 재실행하면, 데이터는 보존된다. 

많은 어플리케이션에서 이런 상황은 문제가 된다. 그렇다면 컨테이너를 삭제해도 데이터를 보존할 수 있는 방법은 없을까? 

## Volumes

Volume 는 컨테이너에 mount(=mapped) 되는 호스트 머신 하드 드라이브 내부에 있는 폴더이다. 

![Untitled](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6545bbde-4b86-4c4e-9925-85c5ea8b6e0e)

`COPY` 명령어랑 다름! `COPY`는 단순히 일회성 snapshot이다. Image가 만들어진 후에는 더 이상 관여하지 않는다. 

**Volume** 은 컨테이너 밖의 폴더와 컨테이너 내부의 폴더를 연결시킬 수 있다. 두 폴더의 변경 사항은 다른 폴더에 반영된다.  따라서 호스트 머신에 파일을 추가하면 컨테이너 내부에서 액세스 할 수 있고,  컨테이너가 매핑된 경로에 파일을 추가하면 컨테이너 외부 (호스트 머신)에서도 사용할 수 있다. 

Volume은 컨테이너가 종료되어도 계속 존재한다.  컨테이너가 다시 시작되면, volume내부에 있는 data는 그 컨테이너 안에서 다시 사용할 수 있다. 컨테이너는 volume 에서 데이터를 read & write 할 수 있다. 

**그렇다면 컨테이너에 어떻게 Volume 을 추가할 수 있을까?**

Dockerfile 에 명령어를 추가할 수 있다. 기존의 dockerfile 에  `VOLUME ["/app/feedback"]` 를 추가해줬다. 

```Dockerfile
FROM node:14  
  
WORKDIR /app  
  
COPY package.json .  
  
RUN npm install  
  
COPY . .  
  
EXPOSE 80  
  
VOLUME ["/app/feedback"]  
  
CMD ["node", "server.js"]
```

다시 이미지를 빌드하고, 컨테이너를 실행해보자.

음... 내용을 입력하니 계속 돌아가기만 하고 저장되지 않는다. 문제를 보기 위해 `docker logs feedback-app` 을 입력해보자. 아래와 같은 에러가 뜬다. 

<img width="700" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/579b4e12-6dc8-4206-a53f-779a1626c219">

```text
(node:1) UnhandledPromiseRejectionWarning: Error: EXDEV: cross-device link not permitted, rename '/app/temp/awesome.txt' -> '/app/feedback/awesome.txt'

```

위 에러를 보면, temp 폴더에서 feedback 폴더로 옮기려 할 때 문제가 발생한다. 이것은 server.js 코드에서 rename 을 사용할 때 생긴 문제이다. 

```javascript
app.post('/create', async (req, res) => {  
  const title = req.body.title;  
  const content = req.body.text;  
  
  const adjTitle = title.toLowerCase();  
  
  const tempFilePath = path.join(__dirname, 'temp', adjTitle + '.txt');  
  const finalFilePath = path.join(__dirname, 'feedback', adjTitle + '.txt');  
  
  await fs.writeFile(tempFilePath, content);  
  exists(finalFilePath, async (exists) => {  
    if (exists) {  
      res.redirect('/exists');  
    } else {  
      await fs.rename(tempFilePath, finalFilePath);  
      res.redirect('/');  
    }  
  });  
});
```

위의 코드를 아래와 같이 변경해보자. rename 하는 대시 copyFile 을 이용해서 복사하고, unlink 를 사용해서 수동으로 삭제했다. 

```javascript
app.post('/create', async (req, res) => {  
  const title = req.body.title;  
  const content = req.body.text;  
  
  const adjTitle = title.toLowerCase();  
  
  const tempFilePath = path.join(__dirname, 'temp', adjTitle + '.txt');  
  const finalFilePath = path.join(__dirname, 'feedback', adjTitle + '.txt');  
  
  await fs.writeFile(tempFilePath, content);  
  exists(finalFilePath, async (exists) => {  
    if (exists) {  
      res.redirect('/exists');  
    } else {  
      await fs.copyFile(tempFilePath, finalFilePath);  
      await fs.unlink(tempFilePath)  
      res.redirect('/');  
    }  
  });  
});
```

다시 이미지를 빌드하고 컨테이너를 실행하면, 잘 된다. 이제 컨테이너를 삭제하고 재실행해도 데이터가 남아있는지 테스트해보자.

http://localhost:3000/feedback/hungry.txt

여전히 안된다 ...!! 왜 안되는지 알아보자.

## Two Types of External Data Storages
 
| Volumes | Bind Mounts |
| ---- | ---- |
| Managed by Docker | Mananged by me |
| Anonymous Volumes / Named Volumes |  |
| 도커는 호스트 머신에 나에게 알려지지 않는 경로의 폴더를 세팅한다. <br>`docker volume` 명령어로 제어된다. | 내가 호스트 머신에 직접 경로를 설정한다. |

### Volumes

**Anonymous Volume** 은 도커에 의해 관리된다. `docker volume ls` 을 입력하면, 현재 있는 모든 volume 들이 나타난다. 만약에 컨테이너를 삭제하면, 이 volume 도 같이 사라진다. 우리가 앞에서 dockerfile 을 사용해서 `VOLUME` 명령어를 통해 생성한 것이 Anonymous Volume 이다.  하지만 이것은 앞에서 봤던 우리의 문제에 전혀 도움되지 않는다. 

<img width="620" alt="Untitled" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a77b88b0-2e3b-4622-a5f9-a8d38c3b6d87">


**Named Volume** 을 사용하면, 컨테이너를 삭제한 후에도 volume 이 유지된다. 새 컨테이너를 시작해도 volume 이 복구된다. 하지만 얘도 도커가 관리하기 때문에, 호스트 머신 어딘가에는 있지만 정확히 어디있는지 내가 알지는 못한다. 즉, **편집하거나 직접 볼 필요가 없는 중요한 데이터에 적합**하다. 이러한 데이터는 실질적으로 호스트 머신의 폴더에 액세스 하지 않을 것이기 때문이다. 하지만 dockerfile 을 이용해서는 Named Volume 을 생성할 수 없다.  그렇다면 Named Volume 은 어떻게 생성할까?

우선, dockerfile 을 다시 아래와 같이 수정하자.

```dockerfile
FROM node:14  
  
WORKDIR /app  
  
COPY package.json .  
  
RUN npm install  
  
COPY . .  
  
EXPOSE 80  
  
CMD ["node", "server.js"]
```

다시 이미지를 빌드하고, 컨테이너를 실행하는데 이때 옵션을 붙이면 된다. 
`docker run -p 3000:80 -d --rm --name feedback-app -v feedback:/app/feedback feedback-node:volumes`

위의 명령어는, -v 를 통해 Named Volume 을 지정한다. 즉, feedback 이라는 이름의 volume 에 /app/feedback 를 저장하라는 의미이다. Anonymous Volume 과 차이점은, 
Named Volume 는 컨테이너가 삭제되어도 없어지지 않는다. 

진짜 그러는지 테스트 해보자. 위의 컨테이너를 통해 [localhost:3000](http://localhost:3000/) 에 접속해서 feedback 을 입력하고, 컨테이너를 종료한 다음 새로운 컨테이너를 만들어서 확인해보자. 

나는 제목에 hungry 를 입력하고, 내용에 i am hungry 를 입력했다. 와우 🥳 드디어 데이터가 날라가지 않고 보존된다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/51c9dc52-d5b5-4d2f-a115-d010488d929f)

### Bind Mounts

소스코드를 변경하면, 이미지를 rebuild 하지 않는 이상 반영되지 않는다. 왜냐면 도커 이미지를 만들 때 스냅샷만 복사하기 때문이다. 하지만 소스 코드를 변경할 때마다 컨테이너 중지하고, 다시 이미지를 빌드하는 것은 너무 번거롭다. 이것을 해결할 방법은 없을까?

이것을 해결하기 위해 **Bind Mounts** 가 나왔다. Volume 은 도커에 의해 관리되고, 호스트 머신 상 어디에 위치해 있는지 우리는 알지 못한다. 반면에 Bind Mounts 를 사용하면 우리가 위치를 알 수 있다. 왜냐면 우리가 직접 경로를 설정하기 때문이다.  따라서 **지속되어야 하고, 편집할 수 있어야 하는 데이터를 저장하는데 적합**하다. 

Bind Mount 도 dockerfile 을 통해 설정할 수 는 없다. 왜냐면 이것은 컨테이너에만 영향을 주는 것이지, 이미지에는 영향이 없다. 아까처럼 터미널에서 옵션을 통해 설정해줘야 한다.  `docker run -p 3000:80 -d --rm --name feedback-app -v feedback:/app/feedback -v "/Users/soeun-uhm/study/Docker-study/data-volumes-01-starting-setup:/app" feedback-node:volumes 
`
이때, 호스트 머신 상에서 **절대 경로** 와 도커 컨테이너 를 매핑해줘야 한다. 
- `/Users/soeun-uhm/study/Docker-study/data-volumes-01-starting-setup` : 내 컴퓨터(호스트 머신)에서 루트 폴더의 절대 경로
- `/app` : 컨테이너 상의 경로 

만약에 절대 경로를 복사하고 싶지 않으면, `-v $(pwd):/app` 를 사용할 수 도 있다 (mac/linux 기준).  Window 에서는 `-v "%cd%":/app` 를 사용하면 된다. 

하지만 도커 컨테이너를 실행하면, 아래와 같은 오류가 뜰 것이다. 나의 경우에는 local host에서 응답없고, docker ps 시 아무것도 안나타났다. 

<img width="630" alt="Untitled" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/928ba6f2-522b-4bfe-ab24-e6645819be34">

`RUN npm install` 로 모든 dependencies 설치한 것 같은데 왜 필요한 module 이 없다는 문제가 뜰까?

왜냐면 dockerfile 에서 npm install 로 모든 모듈이 설치되지만, local에서는 npm install 을 한 적이 없기 때문에 필요한 모듈들이 설치되지 않는다. 저 명령어로는 local에 있는 모든 폴더가 container에서 한 작업들을 덮어쓴다. (반대로 container내에 있는 것은 local을 덮어쓸 수 없다. 이게 가능하면 local에 있는 많은 중요한 파일들을 실수로 지워버리는 일이 발생할 수도 있다..) 이것은 어떻게 해결할까?

우선 컨테이너와 volume 이 어떻게 상호작용하는지 알아보자. 

![Untitled](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/51f7bf7c-439d-46b5-b56c-a21be418da8c)

Volume, Bind Mount 는 `-v` 를 이용해서 컨테이너에 mount 할 수 있다. 

지금은 dockerfile 명령어를 통해, 컨테이너 내부에 /app 폴더 안에 파일들을 복사했다. 그리고 컨테이너 외부에 있는 로컬 호스트 머신의 이 폴더에 파일과 폴더들이 있다.  도커는 호스트 머신의 로컬 파일을 덮어쓰지 않는다. 대신 로컬 호스트 폴더와 그 안에 있는 컨텐츠가 도커 컨테이너에 있는 내용을 덮어쓴다. 그 때문에 로컬에 npm module 이 없는 상황이 컨테이너 내부에 덮어쓰여진다. 

이 문제를 해결하려면, 우리는 도커에게 외부에서 덮어쓰지 말아야 할 내부 파일 시스템에 특정 부분이 있다는 것을 말해줘야 한다. 이것은 다른 anonymous volume 으로 해결할 수 있다. `-v /app/node_modules` 를 추가하면 된다. 

`docker run -p 3000:80 -d --rm --name feedback-app -v feedback:/app/feedback -v "/Users/soeun-uhm/study/Docker-study/data-volumes-01-starting-setup:/app" -v /app/node_modules feedback-node:volumes`

( **:** 앞에 로컬 머신 경로가 붙으면 bind mount 이고, : 앞에 경로가 아닌 것이 붙으면 volume 이름으로 취급이 되어 named volume 이 된다. )

앞의 명령어가 어떤 의미일지 천천히 뜯어보자. 우선, 3가지 종류의 volume 이 컨테이너에 마운트 되어 있다.
- `-v feedback:/app/feedback` -> named volume 
- `-v "/Users/soeun-uhm/study/Docker-study/data-volumes-01-starting-setup:/app"` -> bind mount
- `-v /app/node_modules` -> anonymous volume

named volume 은 컨테이너가 삭제되어도 데이터는 살아있어야 하기 때문에 필요하다. 이때 경로는 지정하지 않으며, 편집될 필요가 없는 데이터에 적합하다. 

bind mount 는 앞에 절대 경로를 지정해주어야 한다. 살아있어야 하고, 편집이 필요한 데이터에 적합하다. 

anonymous volume 은 dockerfile 의 `npm install` 에 의해 생성되는 node_modules 폴더를 보호하기 위해 있다. 이게 어떻게 살아있냐면, 도커는 더 길고 더 구체적인 경로를 채택한다는 규칙이 있다. 즉 `/app/node_modules` 폴더는 살아남고, 외부에서 들어오는 유입 (`/Users/soeun-uhm/study/Docker-study/data-volumes-01-starting-setup` 내에 있는 폴더들) 과 공존한다. 즉, `/app/node_modules` 는 존재하지 않는 로컬의 node_modules 폴더를 덮어 쓴다. 

Bind mount 에서는, 코드를 수정한 후 저장하고, localhost 를 새로 고침하면 이미지를 다시 빌드할 필요 없이 수정사항이 즉각적으로 반영된다. Bind mount 를 사용하면 호스트 머신의 특정 폴더를 컨테이너의 경로와 매핑하기 때문에, 로컬에서 만든 수정사항이 컨테이너에 즉각적으로 반영된다. 


## Read Only Volume

컨테이너 안에서 실행되는 어플리케이션이 write 를 하지 못하게 해야 하는 경우가 생길 수 있다. 이때 bind  mount 를 읽기 전용 볼륨으로 전환할 수 있다. 이것은 bind mount 의 /app 뒤에 `:ro` 를 추가하면 된다. 

` docker run -p 3000:80 -d --rm --name feedback-app -v feedback:/app/feedback -v "/Users/soeun-uhm/study/Docker-study/data-volumes-01-starting-setup:/app:ro" -v /app/node_modules feedback-node:volumes`

이렇게 하면 도커가 이제 그 폴더나 그 하위 폴더에 쓸 수 없게 된다. 우리는 여전히 호스트 머신내에 파일을 쓸 수 있지만, 컨테이너와 컨테이너에서 실행되는 어플리케이션은 read-only 가 된다. 

하지만 위의 명령어는, feedback 폴더나 temp 폴더도 write 를 제한한다. 왜냐면 전체 프로젝트 루트를 bind mount 했고, ro 로 지정했기 때문이다. 

<img width="367" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/57787e47-5352-4916-87b4-4b73f6976fc9">

하지만 feedback , temp 폴더는 소스 코드 내부에서 write 하려는 폴더이다. 따라서 여기에 write 가 가능한지 체크해야 한다. 이때 로직은 아까 익명 볼륨을 생성한 로직과 같다. 도커는 더 구체적인 경로를 채택한다 ! 따라서 새로운 하위 볼륨을 추가해준다. 그러면 이 하위 볼륨은 메인 볼륨보다 우선시된다. 이미 `v feedback:/app/feedback`  는 있으므로, `-v /app/temp` 를 추가해주었다. 

` docker run -p 3000:80 -d --rm --name feedback-app -v feedback:/app/feedback -v "/Users/soeun-uhm/study/Docker-study/data-volumes-01-starting-setup:/app:ro" -v /app/temp -v /app/node_modules feedback-node:volumes`

따라서 temp, feedback 폴더 모두 writable 이 되었다. 

## Mananging Volumes

`docker inspect <Volume>` 를 하면, 볼륨에 대한 상세 정보를 볼 수 있다. named volume 인 feedback 을 검사해 보자. Mountpoint 까지 볼 수 있다. 하지만 이 Mountpoint 는 실제 내 호스트 머신 안에 있는 경로가 아니다. 내 호스트 머신 시스템 상에 도커가 설정한 작은 가상 머신 내부에 있다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e6bb118b-e6a7-4923-a7b0-f9772ccc8692">

## Arguments & Environment Variables

- Arguments(ARG) : 도커파일 내부에 있고, CMD 또는 어플리케이션 코드로 접근할 수 없다.  이미지를 빌드할 때 `--build-arg` 로 지정한다. 
- Environment variables(ENV) : 도커 파일 내부에 있고, 어플리케이션 코드에서 접근할 수 있다. 도커파일 내에서 `ENV` 로 설정하거나, docker run 에서 `--env`로 설정할 수 있다. 

### Environment Variables

마치 환경 변수와 같은 개념이다.  아래 server.js 코드에서, 포트 번호를 환경 변수로 설정하고 싶다고 하자.

```javascript
app.listen(process.env.PORT);
```

이렇게 변경한 후, 도커파일에 `ENV` 를 추가해주면 된다. EXPOSE 명령어 뒤에도, 값을 하드코딩 하는 대신 $PORT 를 통해 환경 변수를 사용할 수 있다.

```Dockerfile
FROM node:14  
  
WORKDIR /app  
  
COPY package.json .  
  
RUN npm install  
  
COPY . .  
  
ENV PORT 80  
  
EXPOSE $PORT  
  
CMD ["node", "server.js"]
```

 도커파일에서 PORT에 지정해준 80 은 기본 값이다. 반드시 이 값이어야 할 필요는 없다. docker run 명령어를 입력할 때, `--env` 또는 `-e` 옵션을 통해 다른 것으로 지정할 수 도 있다.  아래는 PORT 를 8000으로 변경한 것이다. 이때, 이미지를 다시 빌드할 필요없다. 

`docker run -p 3000:80 -d --rm --env PORT=8000 --name feedback-app -v feedback:/app/feedback -v "/Users/soeun-uhm/study/Docker-study/data-volumes-01-starting-setup:/app:ro" -v /app/temp -v /app/node_modules feedback-node:volumes`

환경 변수가 포함된 파일(.env) 를 지정할 수 있다. 그러면 docker run 할 때, `--env-file ./.env` 를 지정하면 된다. 

환경 변수에 저장하는 데이터의 종류에 따라, 보안 데이터를 `Dockerfile`에 직접 포함하고 싶지 않을 수도 있다. 그 대신 런타임에만 사용되는 별도의 환경 변수 파일로 이동시면 된다. (즉, `Docker run`으로 컨테이너를 실행할 때). 그렇지 않으면, 값이 '이미지에 포함'되며, 모든 이가 'docker history <이미지>'를 통해, 이 값을 읽을 수 있다.

일부 값의 경우, 이것이 중요하지 않을 수도 있지만, 자격 증명, 개인 키 등의 경우에는 확실히 피해야만 한다!

### Argument

기본 값을 외부에서 설정하고 싶다면 어떻게 할까? `ARG` 를 사용하면 된다.  아래의 도커파일을 보자. ARG 로 DEFAULT_PORT 값을 설정해주었다. 

```Dockerfile
FROM node:14  
  
WORKDIR /app  
  
COPY package.json .  
  
RUN npm install  
  
COPY . .  
  
ARG DEFAULT_PORT=80  
  
ENV PORT $DEFAULT_PORT  
  
EXPOSE $PORT  
  
CMD ["node", "server.js"]
```

이렇게 하면, 같은 이미지를 기반으로 tag 를 달리 해서 port 번호를 다르게 지정해 줄 수 있다. 
- `docker build -t feedback-node:web-app .`  -> 기본 포트 번호 80 
- `docker build -t feedback-node:dev --build-arg DEFAULT_PORT=8000 . ` -> 기본 포트 번호 8000


## Summary 
- Anonymous Volume & Named Volume & Bind Mount 을 지정하는 명령어
    - `docker run -v /app/data …` → Anonymous Volume
    - `docer run -v <NAME>:/app/data` → Named Volume
    - `docker run -v /path/to/code:app/code …` → Bind Mount
- **Anonymous Volume**
    - 하나의 컨테이너를 위해 생성된다. 
    - 컨테이너가 제거될 때 익명 볼륨도 제거된다. 그렇지만 중지, 재시작에는 살아있다. 
    - 컨테이너 간에 공유될 수 없다. 
    - 익명이기 때문에, 재사용될 수 없다. 
    - Dockerfile 의 `VOLUME` 명령어나 `-v /app/data …` 를 통해 생성된 익명 볼륨은 이미 존재하는 특정 데이터를 잠그는데 유용하다. 이러한 데이터가 다른 모듈에 의해 덮어 씌워지는 것을 방지한다.
    - 익명 볼륨은 여전히 호스트 머신에 폴더를 생성한다. 이것은 컨테이너가 실행되는 동안 유지된다. 이것은 도커가 컨테이너 내부에 모든 데이터를 저장할 필요가 없으며 도커가 이 컨테이너 read-write 레이어 내부의 모든 데이터를 관리할 필요가 없다는 것을 의미한다. 그 대신 특정 데이터를 호스트 머신 파일 시스템에 아웃소싱 할 수 있다. 이것은 성능과 효율에 도움이 된다. 
- **Named Volume**
    - Dockerfile 로 생성될 수 없고, 컨테이너를 실행할 때 `-v` 명령으로 생성된다. 
    - 제너럴하게 생성된다 - 즉, 특정한 컨테이너에 묶여있지 않다. 
    - 컨테이너를 중지 / 삭제 하는 것에 살아남는다 →  제거하려면 도커 CLI 를 사용해야 한다. 
    - 컨테이너 간에 공유될 수 있다. 다수의 다양한 컨테이너에 동일하게 명명된 볼륨 하나를 마운트 할 수 있다. 그곳에서 데이터를 컨테이너 간에 데이터를 공유할 수 있다. 
    - 같은 컨테이너 또는 컨테이너 간에 재사용될 수 있다. 
- **Bind Mounts**
    - 호스트 머신에 데이터가 저장되는 위치를 알고 있다. 
    - Bind mount 는 하나의 특정 컨테이너에 국한되지 않는다. 따라서 다수의 컨테이너에 연결할 수 있다. 
    - 컨테니어 중지 / 삭제하는 것에 살아남는다 → 제거하려면 host file system 을 사용해야 한다. (도커 명령어 사용불가 !)
    - 컨테이너 간에 공유될 수 있다. 다수의 다양한 컨테이너에 동일하게 명명된 볼륨 하나를 마운트 할 수 있다.  그곳에서 데이터를 컨테이너 간에 데이터를 공유할 수 있다. 
    - 같은 컨테이너 또는 컨테이너 간에 재사용될 수 있다. 

![Untitled](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/eeb4cc10-c559-4f14-8a23-f8fd09dfbfe4)

어떤 유형의 Mount를 사용하든, Data는 Container 내에서 동일하게 보이며, Container File System의 폴더나 개별적인 파일들로 표시된다. 올바른 Mount유형을 선택할 때 기준이 될 volume, bind mounts, tmpfs mount 간의 가장 큰 차이점은, Data가 Docker Host내에서 **어디에 존재** 하는지 이다.



## Reference
- Udemy, Docker & Kubernetes sec.3