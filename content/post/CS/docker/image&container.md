+++
author = "Soeun"
title = "Docker Image & Container"
date = "2024-01-01T00:10:10+09:00"
summary = "Docker Image , Container 기초와 명령어 정리"
categories = [
    "CS"
]
tags = [
    "Docker"
]
slug = "docker-image-and-containers"
series = ["Docker"]
series_order = 1
+++

## Images & Containers

### Image

이미지는 설계도이다. 코드와 필요한 환경을 모두 포함하고 있다. Dockerfile 을 이용해서 Docker Image 를 만들 수 도 있고, Dockerhub 에 있는 이미지를 pull 해서 사용할 수 있다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0393f69f-f8ac-41f5-9349-866c8c9dbd13">

<img width="528" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/21091776-e240-4313-a788-3c1a5e78a5d9">
위를 보면 docker 내에서 run 한 node와 local 에서 Run 한 node의 version 이 다르다. 즉 로컬 환경과 독립적으로 도커 컨테이너 내에서 독립적으로 노드가 실행된 것이다. 

**Dockerfile 을 이용해서 이미지 만들기** 

```Dockerfile

FROM node 
 
WORKDIR /app
 
COPY . /app

RUN npm install 

EXPOSE 80

CMD ["node","server.js"]

```


이미지는 **읽기 전용**이다. 소스코드를 변경하고, 다시 [localhost](http://localhost) 사이트에 들어가도 변경 사항이 반영되지 않는다! 왜냐면 이미지는 복사한 시점에서 소스코드의 스냅샷을 만든다. 업데이트한 Source code 를 반영하려면 image rebuild 해야 한다. `docker build .` 다시 실행하면 새로운 ID 할당한다. 

### Container

컨데이터는 돌아가는 unit of software 이다.  run 명령어로 실행한다 (`docker run node`). 이미지를 이용해서 빌드 한다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/86e005d7-3fcb-4b25-b8cb-a79d2abafe20">

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8f1aadc3-2965-41de-845a-3a715117939c">

## Image Layer

이미지는 layer based architecture 이다. 처음 dockerfile 을 이용해서 docker build 를 한 후 , 소스 코드를 수정 후 두번째로 `docker build .` 를 하면 굉장히 빨리 되는데, 이것은 이전에 만들어 놓은 cache 를 사용하기 때문이다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/dd958605-3857-4803-9bd2-3d9f18a6c1cf">

변경 사항 이후에 모든 Instruction 은 모두 다시 실행한다. 위의 Dockerfile 을 다시 보자. 

```Dockerfile
# Base image가져오기 (여기서는 node)
FROM node 

#all the commands 가 app 폴더 내에서 실행된다. 
WORKDIR /app

# which file should go into the image ? 
# COPY . . -> 첫번쨰 점: dockerfile파일을 포함하고있는 폴더 내 모든 파일 , 두번째 점: 그 파일을 저장해야 하는 이미지 내부의 경로 
# (첫번째)Host file system , (두번째)Image/container file system  
COPY . /app

RUN npm install 

#local 에 실행되고 있는 port 노출 (선택사항)
EXPOSE 80

#RUN <-> CMD: 이미지가 생성될때 위 명령어 실행x, container 가 시작할때 실행함 
CMD ["node","server.js"]

```

소스코드를 변경하면 , `COPY ./app` 이후에 모든 명령어를 다시 실행하지만, 사실 `RUN npm install` 은 다시 실행할 필요가 없다! 이럴때는, 아래와 같이 COPY 를 2번에 걸쳐서 나누어서 해주면 된다. 소스코드만 변경되면, `COPY ./app` 이후의 명령어들만 다시 실행해서 빌드하므로, `RUN npm install` 은 이미 캐시되어 있는 데이터를 사용한다. 

```Dockerfile
# Base image가져오기 (여기서는 node)
FROM node 

#all the commands 가 app 폴더 내에서 실행된다. 
WORKDIR /app

COPY package.json /app

RUN npm install 

COPY . /app

#local 에 실행되고 있는 port 노출 (선택사항)
EXPOSE 80

#RUN <-> CMD: 이미지가 생성될때 위 명령어 실행x, container 가 시작할때 실행함 
CMD ["node","server.js"]

```

## Managing Images & Containers

**Container 중지 & 재시작** 

- `docker run` : 새로운 docker 를 시작

	<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d74afdcd-408d-484d-aee3-4377899ac6ef">
	docer run 은 아래 새로운 명령어 입력창이 뜨지 않는다. 즉, Attached 되어 있다. 이것은 container 의 output 을 듣고 있는 상태인 것이다 (log들이 콘솔에 출력된다). 

- `docker start <container ID>` : run 했던 conatainer 재시작

	<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ed565bc7-a850-414b-801f-46a0e65d9b95">

	docker start 를 입력하면, 실행되고 새로운 명령어 창이 뜬다. 즉, detached 되어 있다 (log 가 콘솔창에 뜨지 않는다). detached 된 것을 다시 attach 할 수 도 있다 -> `docker attach <container ID>` . `docker logs` 를 통해 출력된 과거 log 들을 볼 수 ㅣ있다. 과거의 로그, 앞으로의 로그들 모두 보고 싶으면 `docker logs -f <contaier ID>`
	를 입력하면 된다. 

**이미 실행중인 container 에 연결하기**

디폴트로 `-d` 없이 컨테이너를 실행하면, "attached" 모드로 실행된다. 

**Interactive Mode 로 들어가기**

- `docker run -it <Container_ID>` : interact 할 수 있게 만들어줌 (input 입력 가능!)
- `docker start -a -i <Container_ID>` : restart 하면서 Input 을 입력받을 수 있게 해줌

**Image & Container 삭제하기**

- `docker rm <Container_ID1> <Container_ID2> <Container_ID3>`
- `docker images` : 가지고 있는 image 표시
- `docker rmi <IMAGE ID>`: image 제거
- 이미지를 없애기 전에 Container 를 없애야 한다.
- `docker image prune`: 안쓰는 image 모두 제거

**중지된 Container 자동 제거하기**

- `docker run -p 3000:80 -d --rm <IMAGE ID>`: docker stop 하면 자동적으로 container 를 remove 한다.
- `docker stop <container name>` → 컨테이너가 중지되면 자동으로 제거된다 !

**이미지 검사**

- `docker inspect <Image ID>` 

	<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1e982219-1910-4bd9-b7f8-3c65dbd92ab8">

## Sharing Images & Containers

이미지를 가지고 있는 누구나, 이미지를 바탕으로 컨테이너를 생성할 수 있다. 그렇다면, 이미지를 공유하면 된다 !

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6f469a41-67e2-4420-87aa-7b5a32663d60)

Github 와 비슷하게, 이미지를  push 하고 pull 할 수 있는 [DockerHub](https://hub.docker.com/) 가 존재한다. DockerHub 는 공식적인 이미지 저장소이다. 여기서 남들이 올려놓은 public 이미지, 또는 official 이미지를 pull 할 수 있다. 

- `docker push <IMAGE NAME>` : DockerHub 에 이미지 올리기
- `docker pull <IMAGE NAME>` : DockerHub 에서 이미지 가져오기 

중요한 것은, 내 레포에 이미지를 올릴 때는 `<username>/<Image name>` 형태로 이미지 이름을 지어야 한다. 

1.  Docker hub에서 새로운 repository 를 만든다.
2. Image 이름을 `<USER NAME>/<REPOSITORY NAME>` 으로 만든다.
   
   `docker build -t soeunuhm/node-hello-world .`

	<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c717a939-19b2-40a9-8e16-25883eb006d4">

3. `docker push soeunuhm/node-hello-world`
   
   <img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f5736498-d2d5-4f64-bab2-6c6f390537d0">
	push 할때는 dockerhub 에 login 되어있어야 하지만, pull은 누구든지 할 수 있다 !!


```Dockerfile
docker build -t myimage #이미지 생성(이름: myimage)
docker run —name mycontainer myimage #컨테이너 시작 (컨테이너 이름: mycontainer)
docker stop mycontainer #컨테이너 중지
```

## Summary

Docker 는 결국 Image & Container 가 가장 중요하다 !! 이미지는 컨테이너를 만들기 위한 템플릿이고, 이미지를 기반으로 다수의 컨테이너를 만들 수 있다. 이미지는 DockerHub 에서 다운 받거나 (`docker pull`) , Dockerfile 을 이용해서 만들 수 있다 (`docker build`). 

Dockerfile 의 구조는 다 비슷비슷하다. Image 는 캐싱을 통해 빌드 속도를 최적화하고, 재사용성을 위해 레이어 기반 구조(1 instruction = 1 layer)이다.  따라서 Dockerfile 을 작성하는 순서도 중요하다.  

<img width="527" alt="Untitled" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8e0b4de5-3135-4645-8c8b-8515d9f1a2a8">

컨테이너는 `docker run IMAGE` 명령어를 통해 실행할 수 있는데, 다양한 옵션을 추가할 수 있다. 어떤 옵션이 있는지는 `docker run -help` 를 통해 볼 수 있다. 

모든 컨테이너를 리스트 하고(`docker ps`) , 제거하고 (`docker rm`) 멈추고 (`docker stop`) 시작할 수 있다 (`docker start`)

이미지 또한 리스트 하고 (`docker images`) , 제거하고 (`docker rmi, docker image prune`) 공유할 수 있다 (`docker push / pull`)

## Reference 
- Udemy, Docker & Kubernetes sec.2