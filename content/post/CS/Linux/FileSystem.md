+++
author = "Soeun"
title = "Linux File System"
date = "2023-09-20"
description = "리눅스에서는 파일을 어떻게 관리하는가?"
categories = [
    "CS"
]
tags = [
    "Linux",
    "운영체제"
]
image = ""
+++

GDSC Mount 실습에서 간단한 cloud storage system 을 구현해보니, 실제 리눅스의 file system 에 대해 더 깊이 알아봐야겠다는 필요가 느껴졌다. 그래서 본 글에서는 Linux File System 이 무엇이고, 어떻게 작동하는지 알아보려 한다 ! 🧐

-----
## File System

> 파일 시스템은 **파일에 이름을 붙이고 저장, 탐색을 위해 파일을 어디에 위치 시킬 것인지 나타내는 체계**이다. 

> 파일시스템(File system)이란 **파일(자료)를 사용자가 쉽게 접근 및 발견 할 수 있도록 운영체제가 시스템의 디스크상에 일정한 규칙을 가지고 보관하는 방식**으로 리눅스 운영체제의 경우에는 파티션을 나누고 정리하는데 주로 사용된다. 

파일을 저장하는데 일정한 규칙이 필요한 이유는 무엇일까 ? 

운영체제가 파일들을 일정한 규칙을 연속적으로 사용하여 디스크의 파티션상에 저장하게 되면 저장장치 내에서 파일 저장을 저장하는게 용이해지고 파일을 검색,관리를 효율적으로 할 수 있다. 리눅스는 대표적으로 ext3, ext4 iso9660, swap, nfs, xfs등의 파일 시스템을 사용하고 있다.

## Files
### 파일에 대한 정의

> On a UNIX system, everything is a file ; if something is not a file, it is a process.

유닉스 기반인 리눅스 시스템에서도 파일과 폴더를 구분하지 않고, 모두 파일로 정의한다. 다만 다른점은 폴더는 그 디렉토리에 속해있는 파일의 이름을 저장하고 있는 파일로 정의한다. 프로그램, 서비스, 텍스트, 이미지 ... 도 모두 파일이다. 

모든 파일들을 구조적으로 관리하기 위해서, 사람들은 하드 디스크에 트리 구조를 생각하기 쉽다. 큰 브랜치는 여러 브랜치로 갈라지고, 맨 마지막에 잎이 파일이 되는 것이다. 나도 이 이미지로 생각했었는데, 이것이 완전히 정확한 이미지는 아니다. 

### Sort-of files
모든 것을 파일로 정의하는 리눅스이지만, 몇 가지 예외가 있다. 
- Directory : 다른 파일들의 리스트를 저장하고 있는 파일
- Special file : input / output 을 위해 사용되는 파일. 이 파일은 /dev 디렉토리 내에 있다.
- Link : 시스템의 파일 트리 내 여러 부분에서 파일 또는 디렉토리를 볼 수 있게 하는 시스템
- (Domain) sockets : 특별한 파일의 종류인데, TCP/IP socket 과 비슷하다. 파일 시스템에 대한 Inter-process networking 을 제공한다. 
- Named pipes : socket 과 유사한 역할을 수행하는데, 네트워크 소켓 문법을 사용하지 않고 프로세스가 서로 소통하는 방식이다. 

중요한 파일과 디렉토리를 보기 전에, 파티션 (partitions)에 대해 좀 더 알아야 한다.

## About partitioning

### Why partition?

파티션은 하나의 물리 저장장치를 시스템 내부에서 여러 디스크 공간으로 나누는 작업이다. 파티션을 나누는 이유는 데이터가 날라갈 것을 대비해서 안전 장치를 마련해두는 것이다. 하드 디스크를 파티셔닝 하면, 데이터는 분리되어서 저장된다. 만약 사고가 일어나면 하나의 파티션만 날라가고, 다른 파티션에 있는 데이터는 무사할 가능성이 있다. 

공간은 물리적으로, 논리적으로도 나눌 수 있다. 물리적으로 나눈 공간을 Primary, 논리적으로 나눈 공간을 Extended 라고 부른다. 

### Partition Layout and Types
리눅스 시스템에는 두 가지 파티션이 있다 :
- data partition : 보통 리눅스 시스템 데이터, 
- swap partition : 컴퓨터 물리적 메모리의 확장. 

리눅스 시스템은 설치시에 fdisk 를 사용해서 파티션 타입을 정한다. 표준 리눅스 파티션은 swap 은 82번, data는 83번 을 사용한다. 

루트 파티션 (/ 로 표기)는 100-500MB 정도인데, 시스템 config 파일, 가장 기본적인 명령어, 서버 프로그램, 시스템 라이브러리, 유저를 위한 홈 디렉토리를 포함한다. 

- Swap Partition

	swap space 는 시스템만 접속할 수 있고, 정상적으로 작동시에는 볼 수 없다. 스왑은 어떤 일이 일어나든 내가 계속 작업을 할 수 있게끔 보장해주는 시스템이다. 리눅스에서는 'Out of memory, please close applications first and try again' 이라는 명령어를 사용자는 볼 일이 없는데, 왜냐하면 스왑으로 인한 추가 메모리 덕분이다. 
	
	하드 디스크에 있는 메모리를 사용하는 것은 실제 메모리 칩을 사용하는 것보다 느리지만, 이것을 가지고 있다는 것 자체가 굉장한 안정감을 준다. 

- Data Partition

	하드 디스크의 나머지 부분은 data partition 으로 구성되어 있다. 하지만 시스템에 중요하지 않은 모든 데이터가 하나의 파티션에 있을 수 도 있다. 중요하지 않은 데이터가 파티셔닝 될 땐, 일반적으로 설정된 패턴에 따라 분리된다 :
	- a partition for user programs (/usr)
	- a partition containing the users' personal data (/home)
	- a partition to store temporary data like print- and mail queues (/var)
	- a partition for third party and extra software (/opt)

	파티션이 한번 만들어지면, 더 추가할 수 는 있다. 하지만 존재하는 파티션의 사이즈나 속성을 변경하는 것은 가능하지만 추천하지 않는다. 

- 서버 상에서는, 시스템 데이터와 유저 데이터를 분리한다. 아래 시스템들에 대해서는 파티션이 생성된다 :
	- a partition with all data necessary to boot the machine
	- a partition with configuration data and server programs
	- one or more partitions containing the server data such as database tables, user mails, an ftp archive etc.
	- a partition with user programs and applications
	- one or more partitions for the user specific files (home directories)
	- one or more swap partitions (virtual memory)

### Mount Points
모든 파티션은 mount point 를 통해 시스템과 연결된다. mount point 란 파일 시스템 상에서특정 데이터가 있는 위치이다. 일반적으로 모든 파티션은 root partition 을 통해 서로 연결된다. 파티션 위에서, 슬래시(/)로 표기되는 디렉토리가 만들어진다. 빈 디렉토리가 거기에 연결되는 파티션의 시작점이 된다. 

ex) 다음 디렉토리들을 담고 있는 파티션이 있다고 하자. (videos/  cd-images/   pictures/)

우리는 이 파티션을 /opt/media 라는 디렉토리에 붙이고 싶다. 이것을 하기 위해서는 우선 /opt/media 라는 디렉토리가 존재하는지 확인해야 한다. 그리고 mount 명령어를 사용해서, 시스템 관리자는 이 파티션을 시스템에 연결할 수 있다. 

내 시스텀에서 파티션과 mount point 에 대한 정보는 df 명령어를 통해 알 수 있다. 

<img width="852" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/00bdd348-1212-4e64-9c34-bb796c36614b">

## File System Layout
### Visual
리눅스 파일 시스템은 흔히 tree 구조로 생각한다. 

<img width="382" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/36038e22-8e77-4014-926a-ff7867e15142">

가장 위에있는 / 가 바로 root directory 이다. 나도 / 으로 이동한 다음 ls 를 하면 root directory 바로 아래 있는 폴더들을 볼 수 있다.

<img width="573" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/13d882ec-eb35-462f-9fd5-c29cbcee2357">

현재 내가 있는 디렉토리가 어느 파티션에 있는지 보려면, `df -h .` 명령어를 통해 볼 수 있다. 내 디렉토리가 어느 파티션에 있고, 현재 파티션에 얼만큼이나 공간이 남아있는지 알 수 있다.

<img width="591" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7f18f6e9-609b-4bee-b472-31b083f89fea">

### File systems in reality

실제 컴퓨터는 tree 구조에 대해 아무것도 모르고, 이해하지 못한다. 각 파티션은 자기만의 파일 시스템을 가지고 있다. 파일 시스템 내에서, 파일은 inode 로 나타낼 수 있다. inode 는 파일을 실제 구성하는 데이터(파일이 하드 디스크 내 어디에 있는지, 어느 디렉토리 안에 있는지) 를 모은 serial number 같은 개념이다. 

- Inode에 들어있는 정보
	- Owner and group owner of the file.
	- File type (regular, directory, ...)
	- Permissions on the file 
	- Date and time of creation, last read and change.
	- Date and time this information has been changed in the inode.
	- Number of links to this file (see later in this chapter).
	- File size
	- An address defining the actual location of the file data.

inode 에 포함되어 있지 않는 유일한 정보는 파일 이름과 디렉토리이다. 이것은 따로 특별한 디렉토리 파일에 저장되어 있다. 파일 이름과 inode 숫자들을 비교하면서, 파일 시스템은 사용자가 이해하는 대로 트리 구조를 만들 수 있다. `ls -i` 명령어를 통해 inode 숫자들을 볼 수 있다.

<img width="545" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/dbd0f230-c48c-4780-88b5-178d91537245">



## Reference
- https://www.javatpoint.com/linux-file-system
- https://tldp.org/LDP/intro-linux/html/sect_03_01.html
- https://tldp.org/LDP/intro-linux/html/intro_10.html