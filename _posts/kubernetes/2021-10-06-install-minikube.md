---
title: "minikube 설치 및 사용"
excerpt: "minikube 를 설치하고 간단한 사용법을 알아보자"
categories:
  - Kubernetes
tags:
  - Kubernetes
  - Devops
---

쿠버네티스(Kubernetes)를 사용하기 위해 가장 먼저 해야할 일은 클러스터(cluster) 구축이다.  
클러스터란 Control plane과 노드(node)들의 집합이며 쿠버네티스 운영의 가장 큰 단위라고 생각하면 되겠다.  
한개의 클러스터에 서로 연관없는 서비스들을 다양하게 운영한다면 관리나 실수의 문제가 생길 여지가 높으므로 일반적으로 각각의 서비스 운영은 각각의 클러스터를 구축하여 운영하는 것이 일반적이다.  

# minikube

위에서 설명한 클러스터를 구축하는 방법에는 여러가지가 있다.  
그 중 local 환경에 가장 쉽게 클러스터를 구축할수 있게해주는 도구인 minikube에 대해서 살펴보겠다.  

## 설치 전 준비사항

- **Docker 설치** - minikube에서는 minikube를 설치 및 사용하기 위한 환경으로 Docker를 가장 추천한다.  
따라서 minikube 설치 전에 Docker를 반드시 설치하여 사용하는 것이 좋다.  
- **kubectl 설치** - kubectl은 kubernetes의 cluster와 통신하여 다양한 object들의 상태확인 또는 CRUD 작업 등을 위해 사용되는 CLI 도구이다. minikube 설치 후 kubernetes cluster와의 작업을 위해 [여기](https://kubernetes.io/ko/docs/tasks/tools/){:target="_blank"}에 나와있는 방법대로 설치해준다. 

## 설치

minikube는 <https://minikube.sigs.k8s.io/docs/start/>{:target="_blank"} 에서 아래와 같이 본인의 환경에 맞는 minikube 설치 방법을 쉽게 확인할 수 있다.  
![Install minikube]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes/install-minikube/install-minikube.png)  
이게 끝이다! 정말 간단하다.  

## 사용하기

설치를 완료하였으니 이제 minikube 명령어를 사용하는 방법에 대해 알아보자.  

### cluster 시작하기

간단하게 아래의 명령어를 통해 시작할 수 있다.  

```bash
$ minikube start
```

> 혹시 아래와 같은 에러가 발생한다면 minikube가 실행 시 사용할 driver를 찾지 못한 경우이므로 Docker가 정상적으로 설치 되었는지 확인해보길 바란다.  
>```bash
>😄  minikube v1.23.2 on Darwin 11.4
>👎  Unable to pick a default driver. Here is what was considered, in preference order:
>    ▪ docker: Not installed: exec: "docker": executable file not found in $PATH
>    ▪ hyperkit: Not installed: exec: "hyperkit": executable file not found in $PATH
>    ▪ vmware: Not installed: exec: "docker-machine-driver-vmware": executable file not found in $PATH
>    ▪ parallels: Not installed: exec: "prlctl": executable file not found in $PATH
>    ▪ virtualbox: Not installed: unable to find VBoxManage in $PATH
>    ▪ podman: Not installed: exec: "podman": executable file not found in $PATH
>
>❌  Exiting due to DRV_NOT_DETECTED: No possible driver was detected. Try specifying --driver, or see https://minikube.sigs.k8s.io/docs/start/
>```

minikube는 docker image를 다운받고 실행하여 kubernetes cluster를 생성하고, kubectl과의 연결을 위해 `~/.kube/config` 파일 setting까지 완료해준다.  
하나씩 살펴보자면, 아래와 같이 확인이 가능하다.  

```bash
# minikube 시작 시 download 되는 docker image
$ docker images
REPOSITORY                    TAG       IMAGE ID       CREATED       SIZE
gcr.io/k8s-minikube/kicbase   v0.0.27   9fa1cc16ad6d   2 weeks ago   1.08GB
```

```bash
# 다운받은 docker image로 minikube container를 실행하여 kubernetes cluster 생성
$ docker ps
CONTAINER ID   IMAGE                                 COMMAND                  CREATED          STATUS          PORTS                                                                                                                                  NAMES
1d1d7452e46b   gcr.io/k8s-minikube/kicbase:v0.0.27   "/usr/local/bin/entr…"   16 minutes ago   Up 16 minutes   127.0.0.1:50210->22/tcp, 127.0.0.1:50211->2376/tcp, 127.0.0.1:50213->5000/tcp, 127.0.0.1:50214->8443/tcp, 127.0.0.1:50212->32443/tcp   minikube
```

```bash
# kubernetes cluster와 kubectl의 연결을 위한 정보가 담긴 config 파일 생성
$ cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/name/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Mon, 04 Oct 2021 00:50:31 KST
        provider: minikube.sigs.k8s.io
        version: v1.23.2
      name: cluster_info
    server: https://127.0.0.1:50214
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Mon, 04 Oct 2021 00:50:31 KST
        provider: minikube.sigs.k8s.io
        version: v1.23.2
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/name/.minikube/profiles/minikube/client.crt
    client-key: /Users/name/.minikube/profiles/minikube/client.key
```

이렇게 kubernetes cluster가 손쉽게 생성이 되었고 아래의 명령어를 통해 정상적으로 동작하고 있음을 확인할 수 있다.  

```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:50214
CoreDNS is running at https://127.0.0.1:50214/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

### cluster 일시정지/재가동

```bash
$ minikube pause
```

kubernetes cluster를 일시정지하는 명령어로 일시정지 후에는 kubectl을 통한 cluster의 접근이나 추가적인 명령어는 timeout이 발생하며 사용이 불가능하다.  
단, 이미 배포된 application에는 영향을 미치지 않기 때문에 계속 동작한다.  

일시정지 상태에서 다시 cluster를 재가동 하고자 할때는 아래와 같은 명령어를 사용한다.  

```bash
$ minikube unpause
```

### cluster 종료

```bash
$ minikube stop
```

kubernetes cluster를 위해 구동되었던 minikube docker container가 종료되며 cluster가 종료되게 된다.  
또한 cluster와 kubectl의 연결을 위해 사용되었던 `~/.kube/config` 파일의 내용도 모두 사라진다.  
단, 따로 저장되어있는 cluster 내의 생성했던 리소스들의 정보는 삭제되지 않기 때문에 다시 `minikube start` 명령어를 통해 cluster를 생성하면 기존에 생성해놨던 application 같은 리소스들이 동일하게 생성되어 구동된다.  
즉 `stop` 명령어는 추후 minikube의 restart를 위하여 사용되는 명령어이다.  

### cluster 삭제

```bash
$ minikube delete
```

`stop`을 통한 cluster의 종료는 저장되어 있는 생성된 리소스들의 정보를 삭제하지 않아 다시 시작할때 복원을 해주지만, `delete`를 통해 cluster를 종료한다면 이러한 정보들까지 모두 삭제한다.  
따라서 `delete` 후 다음번 minikube를 시작시 완벽히 초기상태로 시작이 된다.  

<br>
## 마무리

이번 글에서는 local 환경에서 가장 손쉽게 kubernetes cluster를 구축할 수 있게해주는 minikube에 대해 알아보았다.  
사용법이 굉장히 간단하고 local 환경에서 kubernetes 관련 작업들은 다양하고 쉽게 테스트 해볼 수 있으니 반드시 사용해보길 추천한다.  

<br>
<br>
**[참고]**  

<https://minikube.sigs.k8s.io/docs/start/>{:target="_blank"}
