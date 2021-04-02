---
date: 2021-03-28 02:29:39
layout: post
title: "Docker-Concept"
subtitle:
description:
image:
optimized_image:
category:
tags:
- docker
- container
author: newvelop
paginate: false
---
도커를 진행하기에 앞서 도커란 무엇인가에 대해 살펴본다.

### 도커란
도커라는 단어에 대해서 여러번 들어봤지만 다시 한 번 이것이 무엇인지 정의하고 학습을 진행하도록 한다. 도커란 '컨테이너 기술을 사용하여 프로그램을 쉽게 배포하고 운영할 수 있게 해주는 오픈소스 플랫폼'이라고 생각할 수 있다.

#### 도커 특징
도커의 특징을 다른 사이트를 참고하여 정리를 해봤다
- 쉬운 확장성 : 컨테이너들이 가볍기 때문에 쉽게 확장할 수 있음
- 쉽고 빠른 설정 : 몇가지 문서만 작성하고 커맨드라인에 옵션을 부여하면 기존 운영의 복잡한 설정을 한 번에 해결할 수 있음
- swarm : 도커에서 swarm이란 것을 제공하는데 이는 같은 서버의 클러스터링을 쉽게 할 수 있게 해준다. 이는 차후 살펴보는걸로
- routing mesh : 외부에 노출 시키는 포트랑 컨테이너의 열린 포트 매핑을 할 수 있으며, 컨테이너 끼리 열린 포트를 통해 서로 통신을 할 수 있다.
- 보안성 관리 : 자체적으로 security 설정을 관리할 수 있는 기능을 제공한다.

#### 컨테이너
위에서 언급했듯이 컨테이너 기술을 이용했다고 하는데 이 컨테이너란 무엇인가를 짚고 넘어갈 필요가 있다. 이 컨테이너라고 함은 리눅스에서 많이 접할 수 있는데 리눅스 컨테이너는 단일 리눅스 시스템에 동작하고 프로세스를 격리시켜서, 각 프로세스마다 독자적인 리눅스 시스템 환경을 구축하는 것이라고 한다. 즉 OS의 커널은 프로세스마다 공유를 해서 프로세스마다 커널이 존재하는 것은 아니지만, 프로세스별로 컨테이너가 되어서 각자 시스템 환경을 구축하고 있다는 소리이다. 이를 단순히 표현하자면 OS 수준에서의 가상화라고 한다.

그렇다고 해서 리눅스 컨테이너와 도커의 컨테이너 개념이 같냐 라고 묻는다면 그건 다르다.

#### LXC VS 도커
1. 구조적 차이
  리눅스의 컨테이너는 OS와 거의 동일하게 동작을 하기 때문에, 여러 어플리케이션이 하나의 컨테이너에 올릴 수 있다. 반면에 도커는 하나의 어플리케이션을 하나의 컨테이너로 사용하게끔 설계되어있다. 
  ![screensh](../assets/img/2021-03-28-Docker---Concept/lxc_vs_docker.png)

2. 데이터 저장 여부
  리눅스의 컨테이너는 독자적인 시스템 환경이라고 언급했듯이, 해당 컨테이너의 프로세스에서 사용하는 데이터들은 전부 해당 컨테이너에 저장이 된다. 반면에 도커의 데이터는 컨테이너에 저장되지 않는다. 데이터를 어떻게 유지하냐 라고 하면 도커의 Volume 개념이 있어서 Volume을 컨테이너에 바인드시켜서 데이터를 사용한다.

  ![screensh](../assets/img/2021-03-28-Docker---Concept/Docker-volume.png)

#### 도커 VS VM
도커와 VM의 차이점에 대해서 살펴보면 VM같은 경우는 본인 컴퓨터의 OS위에서 Hypervisor를 이용해 가상화한 OS를 위에 올리고, 이 OS위에 다시 어플리케이션을 올리는 개념이다. 즉 OS 위에 OS가 올라가서 Overhead가 꽤 크다. 반면에 도커의 경우 OS위에 도커 Engine이 올라가고, 이 위에 어플리케이션이 묶인 컨테이너들이 올라가는 구조적 차이가 있다. 즉 OS위에 OS가 올라가는지 안올라가는지의 차이가 있다

![screensh](../assets/img/2021-03-28-Docker---Concept/docker_vs_vm.jpg)

#### 동작 원리
![screensh](../assets/img/2021-03-28-Docker---Concept/Docker-architecture.png)

해당 동작 원리는 도커관련 유튜브 영상을 보고 요약한 부분이다. 구조는 상단의 그림과 같다. 먼저 도커 엔진을 통해서 컨테이너들이 관리가 되는데, 도커 엔진은 클라이언트와 호스트(서버)가 있으며, 이들이 각각 REST API를 통해 통신을 한다. 엔진의 클라이언트는 사용자의 command를 체크해서 커맨드 기반으로 REST API를 호스트에 호출한다. 호스트는 도커 데몬을 실행시키고 이 데몬이 클라이언트의 REST API를 받아서 처리한다. 도커 데몬은 이미지, 컨테이너, 네트워크 등등 도커의 핵심 요소들을 관리한다.

#### 레이어
도커 컨테이너와 이미지에 대한 핵심 기능이다. 도커의 컨테이너는 이미지를 기반으로 프로세스를 실행시키는 개념이라 보면 되는데 이 컨테이너의 기반인 이미지는 여러 레이어들로 구성되어있다.

![screensh](../assets/img/2021-03-28-Docker---Concept/container-layer.png)

위의 그림과 같이 컨테이너는 이미지 레이어들의 위에 레이어 하나를 얹는 걸로 구성되며, 이미지 레이어는 불변이고, 컨테이너 레이어만 컨테이너 상에서 수정할 수 있다. 도커 이미지의 경우, 버전이 변경되었어도 레이어의 일부만 바뀔 경우 바뀐 레이어만 새로 받기 때문에 효율적이며, 사용자가 어플리케이션을 이미지로 만들고자 할 경우 이 점을 고려하여 버전 업데이트 될때 최소한의 레이어만 변경할 수 있게 이미지를 작성하면 좋다.


참고
- https://www.redhat.com/ko/topics/containers/whats-a-linux-container
- https://www.redhat.com/ko/topics/containers/what-is-docker
- https://medium.com/extales/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-linux-containers-lxc-%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-132dde6331f8
- https://laptrinhx.com/docker-vs-lxc-lxd-what-s-the-best-for-your-website-3453457212/
- https://archives.flockport.com/lxc-vs-docker/
- https://chacha95.github.io/2020-08-08-Docker_Kubernetes1/
- https://youtu.be/rOTqprHv1YE
- https://data-flair.training/blogs/docker-features/