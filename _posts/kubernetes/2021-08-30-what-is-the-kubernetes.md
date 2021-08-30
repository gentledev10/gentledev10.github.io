---
title: "쿠버네티스(Kubernetes)란?"
excerpt: "쿠버네티스(Kubernetes)란 무엇인지 알아보자"
categories:
  - Kubernetes
tags:
  - Kubernetes
  - Devops
---

쿠버네티스를 처음 접하는 개발자들의 진입장벽을 없애고자 하는 목적으로 개념과 사용방법들에 대해 최대한 쉽게 글을 써보려한다.  
먼저 쿠버네티스란 무엇인지부터 살펴보겠다.  

<br>
# 쿠버네티스(Kubernetes)란?

[쿠버네티스(Kubernetes)](https://kubernetes.io/){:target="_blank"}는 containerized application 을 관리 해주는 오픈소스 플랫폼이다.  
쉽게 말해보면 요즘 많은 곳에서 사용하는 기술인 container 를 하나하나 일일이 개발자가 직접 관리 하는게 아니라, 쿠버네티스에게 어떤 방식으로 관리를 해라라고 지시만 내리면 실제 관리는 쿠버네티스가 담당하는 방식이다.  
여기에서 얘기한 관리에는 배포방식, scailing, network 등등 굉장하게 다양한 기능들이 포함된다.  
따라서 쿠버네티스는 container orchestration 이라고 불리며 Kubernetes 이름의 "K"와 "s" 사이에 8글자가 있어 K8s 라고도 불린다.  

혹시 아직 잘 와닿지 않는 분들을 위하여 한가지 예를 들어 설명해보겠다.  
특정 Application container 를 5개 운영하며 배포는 Rolling 방식을 사용한다고 해보자.  

"***쿠버네티스의 도움 없이(정확하게 말하자면 container orchestraion의 도움 없이)...***"  
개발자는 container 가 돌아갈 instance 들을 모두 직접 관리해야하고, 5개의 container를 직접 적절한 instance에 배포하며 rolling update 에 대한 부분까지 직접 컨트롤을 해줘야한다.  
생각만해도 귀찮은 일이지만.. 혹시 누군가가 이정도의 요구사항은 어찌저찌 해결했다고 하자.  
하지만 서비스가 커지면서 더 많은 container 들과 instance들을 관리하게되고, 추가적으로 scailing이나 network policy 같은 것들도 설정해야 한다면 어떻겠는가?  
어느순간 관리의 한계를 느끼게 될 것이다.  

"***쿠버네티스를 사용하면...***"  
개발자는 단지 쿠버네티스에게 명령어 또는 YAML 파일을 통하여 container 는 5개를 운영하고 배포는 Rolling 방식을 사용하겠다! 라고 명령만 내려주면 끝이다.  
마찬가지로 서비스가 커져서 Container 갯수가 늘어나던, scailing을 적용하던, 다른 기능들을 추가하던 우리는 단지 쿠버네티스에게 우리가 원하는 환경에 대해 명령만 내려주면 된다.  
우리가 명령을 내린 환경과 일치하도록 쿠버네티스는 쉬지않고 container 환경을 관리해준다.  
결국 이것이 바로 쿠버네티스가 하는 일이다!  

<br>
# 왜 이렇게 유명해졌나?

Docker의 등장으로 container의 사용이 늘어났고, MSA(Microservices Architecture) 또한 많은 곳에서 채택하면서 한개의 큰 서비스를 더욱 작은 단위의 container로 나누는 일들이 많아져 수많은 container를 다루기 위해 필연적으로 container orchestration 도구가 필요하게 됐다.  
Container orchestration 도구는 쿠버네티스 외에도 여려가지가 있는데 ([Docker swarm](https://docs.docker.com/engine/swarm/){:target="_blank"}, [AWS ECS](https://aws.amazon.com/ko/ecs/){:target="_blank"} 등등..) 그 중 쿠버네티스가 현재 de facto standard (사실상 표준) 으로 여겨진다.  

쿠버네티스는 [CNCF](https://www.cncf.io/){:target="_blank"} 에서 관리되고 있는 오픈소스 프로젝트로 Google, Red Hat, AWS 등등 이미 많은 기업에서 참여가 이뤄지고 있으며, 엄청나게 거대한 커뮤니티를 구축하고 있기 때문에 발전의 속도가 빠르다.  
또한 쿠버네티스는 특정 플랫폼에 종속되지 않기 때문에 on-premise 또는 cloud 각각, 또는 혼합하여도 자유롭게 사용이 가능하다.  
AWS, GCP, Azure 같은 곳에서는 이미 managed kubernetes 서비스를 지원하고 있어 더욱 사용이 편리하다.  
이외에도 쿠버네티스 내의 다양하고 자유롭게 사용할 수 있는 많은 기능들, 배포환경을 쉽고 편리하게 관리해주는 [Helm](https://helm.sh/){:target="_blank"} 등등의 이유로 쿠버네티스는 매해 더욱 확고하게 자리를 지키고 있다.  

<br>
# 마무리

쿠버네티스란 무엇인지 간략하게 살펴보았다.  
내부에 많은 기능들이 있는데, 알아야 될 사항들에 대해 다음 글부터 하나하나씩 살펴보도록 하겠다.  
