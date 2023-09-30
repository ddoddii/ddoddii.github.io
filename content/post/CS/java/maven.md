+++
author = "Soeun"
title = "[Java] Maven으로 build 하기"
date = "2023-09-27"
description = "Maven 에 대해 알아보자"
categories = [
    "CS"
]
tags = [
    "Java"
]
image = ""
+++


소프트웨어공학 과제 요구사항에 maven 의 pom.xml 을 통해 dependency 를 관리하라는 것이 있었다. 그래서 이번 글에 maven 이 무엇이고, pon.xml 파일이 어떠한 구조로 이루어져 있는지 알아보겠다. 그리고 제일 삽질 했던 것이 pom.xml 파일에 dependency 를 추가해도 `Cannot resolve symbol` 가 떴는데, 이것을 해결하는 방법에 대해 알아보겠다. 


---- 

## Build , Complie  ?
> Compile : 개발자가 작성한 소스코드를 분석해 기계어(바이너리 코드)로 번역하는 과정이다. 

이러한 과정을 해주는 프로그램을 컴파일러 라고 한다. 자바의 경우, 자바 가상 머신(JVM)에서 실행가능한 바이트코드 형태의 클래스 파일이 생성된다. 즉, .java 라는 파일을 바탕으로 .class 라는 클래스 파일이 생성된다. 

> Build : 소스코드 파일을 컴퓨터에서 실행할 수 있는 독립 소프트웨어 산출물로 변환하는 과정, 결과물이다. 

사용자가 작성한 소스코드 파일 (.java) 를 컴파일해서 (.class) 가 되면, 이것을 컴퓨터가 실행할 수 있는 상태로 변환하는 것을 빌드라고 한다. 즉, 컴파일된 코드를 실제 실행할 수 있는 상태로 만드는 것이다. 

빌드 툴로는 Ant, Maven, Gradle 이 있다. 

- Build = Compile + 그 외 작업
- Run = Build + 실행 

## Maven 이란 ?
Java기반 프로젝트의 **라이프사이클 관리**를 목적으로 하는 빌드 도구이다. 컴파일과 빌드를 동시에 수행, 테스트를 병행하거나 서버 측 Deploy 자원을 관리할 수 있는 환경을 제공한다. 또한 **라이브러리 관리 기**능도 내포하고 있다. Java로 개발하다 보면 다양한 라이브러리를 필요로 하게 되는데, **pom.xml 파일**에 필요한 라이브러리만 적으면 Maven이 알아서 다운받고 설치해주고 경로까지 지정해준다.

* Life cycle      : 논리적인 작업 흐름

* pom.xml       : Project Object Model, 메이븐이 프로젝트를 처리하는 필요한 정보를 제공하는 파일

* Artifact         : 프로젝트에 필요한 jar, war, pom 등등

* Deploy         : 아티페그를 로컬 저장소에 저장하는 행위

여러 명의 개발자가 하나의 프로젝트를 진행할 때, pom.xml 파일을 보고 어떠한 dependency 를 설치했는지 볼 수 있다. 이 pom.xml 파일에 필요한 라이브러리를 정의해 놓으면, 네트워크를 통해서 라이브러리들을 자동으로 다운받아준다.  마치 python project 에서 requirements.txt 를 만드는 것과 같은 개념이다. 

## POM

POM은 "Project Object Model" 이다. Maven project 라면 관련된 모든 내용(dependency 등) 이 모두 xml 형태로 pom.xml 에 저장되어 있다. Apache Maven 공식 문서에는 `'In the Maven world, a project does not need to contain any code at all, merely a pom.xml.'` 라고도 나와있다. 

### pom.xml

아래는 내 프로젝트의 pom.xml 파일이다. 추가하고자 하는 dependency는 <dependencies></dependencies> 안에 추가하면 된다. 

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<!-- Build Settings -->
	<groupId>edu.yonsei.csi3106</groupId>
	<artifactId>homework2</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>

	<name>homework2</name>
	<url>http://maven.apache.org</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.5</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.jacoco</groupId>
				<artifactId>jacoco-maven-plugin</artifactId>
				<version>0.7.7.201606060606</version>
				<executions>
					<execution>
						<goals>
							<goal>prepare-agent</goal>
						</goals>
					</execution>
					<execution>
						<id>report</id>
						<phase>prepare-package</phase>
						<goals>
							<goal>report</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

    <!-- List of dependencies -->
	<dependencies>
		<dependency>
			<groupId>org.junit.jupiter</groupId>
			<artifactId>junit-jupiter-api</artifactId>
			<version>5.3.0</version>
		</dependency>
		<dependency>
			<groupId>org.junit.jupiter</groupId>
			<artifactId>junit-jupiter-engine</artifactId>
			<version>5.3.0</version>
		</dependency>
		<dependency>
			<groupId>org.junit.jupiter</groupId>
			<artifactId>junit-jupiter-params</artifactId>
			<version>5.3.0</version>
		</dependency>
		<!-- Jackson Core -->
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-core</artifactId>
			<version>2.13.0</version>
		</dependency>
		<!-- Jackson Data Binding (for object mapping) -->
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.14.0</version>
		</dependency>
	</dependencies>


</project>

```


## Maven Dependency 동기화 

이렇게 추가하고 싶은 dependency 들을 pom.xml 파일에 추가하고, import 를 했는데도 `Cannot resolve symbol` 이 뜨는 경우가 있을 것이다. 그 경우 Intellij 에서 오른쪽에 있는 maven 을 클릭한 후, 회전하는 화살표의 `Reload All Maven Projects` 를 눌러야 한다. 그러면 정상적으로 import 가 된다 ! 

<img width="1389" alt="스크린샷 2023-09-27 오후 12 21 36" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/23fa926c-859a-4533-ab59-3194e6766e3f">



## Reference
- https://maven.apache.org/pom.html