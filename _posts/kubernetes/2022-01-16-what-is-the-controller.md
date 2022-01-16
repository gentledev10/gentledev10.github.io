---
title: "Kubernetes controller"
excerpt: "Kubernetes controller의 역할과 종류에 대해 살펴보자"
categories:
  - Kubernetes
tags:
  - Kubernetes
  - Devops
---

# Controller

이전 [포스트]({% post_url /kubernetes/2021-12-20-what-is-the-pod %})에서 Pod의 개념과 배포 방법에 대해 알아보았다.  
하지만 일반적으로 Pod를 직접 배포해서 사용하는 경우는 거의 없다고 봐도 무방한데 이유는 그렇게 되면 배포된 모든 Pod를 개발자가 직접 관리해 줘야 하기 때문이다.  
우리가 Kubernetes라는 container orchestration tool 을 사용하는 이유 중 하나는 수많은 container 들을 쉽게 관리하기 위해서인데,  
Pod를 일일이 배포해서 직접 관리를 한다면 그냥 container를 우리가 배포하여 운영하는 것과 전혀 차이가 없다.  

그래서 Kubernetes 에서는 `Controller` 라는 개념으로 Pod의 배포와 관리를 도와 준다.  
필요에 따라 다양한 `Controller`들이 있는데 이번 글에서는 종류와 역할에 대해서만 간단히 설명하고, 다음 포스트부터 하나씩 더 깊게 살펴보겠다.  

## 종류

Controller의 종류와 간단한 특징에 대해 알아보자.  

- **ReplicaSet** - Pod의 집합을 관리한다. ReplicaSet의 선언시 Pod의 갯수, container image 등을 선언해 놓으면 계속 해당 조건을 만족시키도록 Pod를 관리해준다.  
예를들어 nginx container를 가진 Pod 4개를 관리해줘! 라고 요청하면 ReplicaSet은 지속적으로 Pod의 상태를 체크해서 정상적으로 동작하는 4개의 Pod를 유지해준다.  
- **Deployment** - ReplicaSet의 상위 개념으로 ReplicaSet을 관리한다. 즉 ReplicaSet, Pod가 모두 관리된다는 뜻이며 몇가지 Pod update 방식도 제공해주기 때문에 더욱 편리하게 배포 관리가 가능하다.  
따라서 ReplicaSet을 사용하기보단 Deployment를 사용하는 것이 Kubernetes의 공식 권장사항이다.  
예를들어 nginx container를 가진 Pod 4개를 Deployment를 통해 배포하면 해당 Pod를 관리하는 ReplicaSet이 Deployment에 의해 관리된다.(즉 Pod 4개가 ReplicaSet에 의해 유지됨)  
이 상태에서 nginx container의 version을 변경한다면 새로운 ReplicaSet이 생성되어 새로운 버전의 nginx container를 가진 Pod가 생성되고 이전 ReplicaSet이 삭제되며 Pod 또한 삭제된다.  
이렇게 Pod의 관리뿐 아니라 ReplicaSet도 관리를 해주기 때문에 당연히 ReplicaSet보단 Deployment를 사용하는 것이 좋다.  
- **StatefulSet** - 이름에서 유추할 수 있듯이 데이터베이스 등과 같이 상태가 유지되어야 하는 application을 지원하기 위한 controller 이다.  
Application이 종료 되더라도 연결되었던 저장소는 그대로 남아 있어 새로운 Application이 실행되면 기존의 저장소와 연결되어 상태를 유지시켜 준다.  
- **DaemonSet** - Kubernetes cluster 내에 생성된 모든 Node에 대해 배포하고자 하는 Pod가 있다면 DasemonSet을 사용할 수 있다.  
Node가 생성되면 DaemonSet에 정의된 Pod를 생성하고 Node가 사라지면 생성된 Pod도 사라진다.  
개발을 하다보면 여러 Node에 배포된 Application의 Log를 추출해야 하는 경우가 일반적인데,  
이런 경우 Log를 추출해야 하는 agent 역할을 하는 application을 DaemonSet으로 관리하여 각각의 Node에 배포함으로써 모든 Application의 Log를 빠짐없이 추출할 수 있다.  
- **Job** - 계속 running 되는 Pod를 생성하는 것이 아닌, Pod를 생성하여 원하는 작업을 한 후 종료하고자 할 때 사용할 수 있는 controller 이다.  
환경에 따라 다양하게 사용될 수 있으며 Pod 생성 전략, 작업 실패시 전략 등 다양하게 설정을 변경하여 사용할 수 있다.  
- **CronJob** - [Cron 표현식](https://en.wikipedia.org/wiki/Cron){:target="_blank"}을 사용하여 원하는 시점에 바로 위에서 확인한 Job을 생성해 주는 controller 이다.  

<br>
## 마무리

간단하게 다양한 Controller의 종류와 역할에 대해 살펴보았다.  
상황에 따라 필요한 Controller를 자유롭게 쓸 수 있는것을 목표로 다음 글에서 ReplicaSet부터 시작해 하나씩 살펴보도록 하겠다.  

<br>
<br>
**[참고]**  

<https://kubernetes.io/ko/docs/concepts/workloads/controllers/>{:target="_blank"}
