---
title: "Kubernetes Pod란?"
excerpt: "Pod란 무엇인지 살펴보자"
categories:
  - Kubernetes
tags:
  - Kubernetes
  - Devops
---

Pod란 Kubernetes 에서 배포할 수 있는 가장 작은 단위이며, 한개 또는 여러개의 container 들의 묶음 이라고 생각하면 된다.  
즉 Kubernetes cluster 내에 container를 배포하기 위해서는 container 별로 배포를 하는것이 아닌 Pod라는 단위로 container들을 묶어서 배포를 할 수 있다.  
물론 Pod에 대해 처음 공부하려는 사람이라면 위의 말이 이해하기 쉽지 않을 것이다.  
걱정말고 직접 예제와 함께 Pod를 생성해보며 개념과 특징들에 대해 알아본 뒤 위의 정의에 대해 다시한번 읽어 보면 좋을 것 같다.  

## Pod 생성

예제와 함께 Pod에 대해 알아보기 위해 제일 먼저 생성하는 방법에 대해 알아보자.  
기본적으로 Kubernetes 의 Object 생성은 `kubectl` 을 이용하는 방법과 `yaml` 파일을 이용하는 방법이 있다.  
`kubectl`을 이용하는 방법은 편리하긴 하지만 Kubernetes cluster 내의 생성된 다양한 Object 들의 관리가 어렵고 때때로 세부 옵션 설정이 불가능한 경우가 있다.  
따라서 다른 소스코드처럼 파일로 생성되어 관리에 수월한 `yaml` 파일을 통해 생성하는 방법만 다루겠다.  

자 그럼 간단한 Pod를 생성해보자.  
(실습을 위해 minikube로 Kubernetes cluster 를 먼저 시작하자. 혹시 방법에 대해 모른다면 이전 [포스트]({% post_url /kubernetes/2021-10-06-install-minikube %}){:target="_blank"} 참고를 부탁한다.)  

### 한개의 container를 가진 Pod

<script src="https://gist.github.com/gentledev10/10038b052a92d577f3bf0a77d51b917a.js"></script>

먼저 한개의 container를 가진 Pod를 생성하는 예제를 통해 기본적인 생성 방법을 알아보자.  

- `apiVersion` - Pod 생성에 사용할 API version 을 의미하며 여기서는 v1 을 사용하였지만, Pod가 아닌 다른 object 에 따라서는 v1beta1, v1alpha1 같은 version 을 사용하는 경우도 있다.  
- `kind` - 어떤 종류의 object 를 생성할 건지 명시해주며 여기서는 Pod 을 생성함을 알려주고 있다.  
- `metadata` - Object 에서 사용되는 `name`, `namespace` 등등의 다양한 정보를 여기에서 지정해 줄 수 있다. 예제에서는 `metadata`의 `name` field 를 가지고 Pod의 이름을 지정해주고 있다. [여기](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#objectmeta-v1-meta){:target="_blank"}에서 1.22 버전 기준으로 사용할 수 있는 더 다양한 field 값들을 확인할 수 있다.  
- `spec` - Pod이 가질 수 있는 고유의 spec 을 여기에 명시 해 줄 수 있다. 예제에서는 containers field 를 사용하여 Pod 에서 구동할 ubuntu container 에 대한 정의(Ubuntu container에서 1초마다 hello를 출력)를 해주고 있다. 사용할 수 있는 field에 대한 정보 또한 [여기](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#podspec-v1-core){:target="_blank"}에서 1.22 버전 기준으로 확인해 볼 수 있다.  

위의 코드를 `single-container.yaml` 로 저장하고, 다음의 명령어를 통해 드디어 Pod를 배포할 수 있다.  

```shell
$ kubectl apply -f single-container.yaml
pod/single-container-pod created
```

Pod가 정상적으로 배포가 됐는지 확인을 위해 아래의 명령어를 사용해보자.  

```shell
$ kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
single-container-pod   1/1     Running   0          17s
```

명령어의 결과와 같이 우리가 만든 `single-container-pod`란 이름을 가진 Pod가 생성된 걸 확인할 수 있다.  
여기에서 `READY` 의 `1/1`은 1개의 container 중 1개가 ready 상태임을 나타내며 즉 Pod 내의 모든 container 가 정상적으로 작동되고 있음을 확인할 수 있다.  

그렇다면 이번엔 우리가 배포한 Pod 내부의 container의 log 를 확인하는 방법에 대해 알아보자.  
`kubectl logs <Pod 이름>` 명령어를 통해 아래와 같이 실제 container의 log 또한 확인해 볼 수 있다.  

```shell
$ kubectl logs single-container-pod
hello
hello
...
```

마지막으로 Pod를 삭제하고 싶다면 `kubectl delete` 명령어를 이용하자.  
Pod 뿐만 아니라 다른 모든 Object 삭제시 사용가능한 명령어이며, 여기서는 우리가 생성한 Pod 삭제를 위해 아래와 같이 사용이 가능하다.  

```shell
# Pod의 이름으로 삭제 가능
$ kubectl delete pods single-container-pod
```

### 여러개의 container를 가진 Pod

이번 포스트의 맨 위의 설명을 다시 살펴보면 Pod는 한개 또는 여러개의 container들을 가질 수 있다고 나와있다.  
위에서 한개의 container를 가진 Pod를 배포했다면 이번에는 2개의 container를 가진 Pod를 배포해보자. (물론 더 많은 container를 가진 pod의 생성도 가능하다.)  

<script src="https://gist.github.com/gentledev10/f39d9047faf611eec58a40af156e0865.js"></script>

Pod를 생성하는 기본적인 yaml 파일의 내용은 한개의 container를 만들때와 동일하다.  
다만 여러개의 container를 한개의 Pod에서 사용하고 싶을때는 예제와 같이 `containers` array에 여러개의 container 정보를 명시해주면 된다.  

위의 예제에서는 hello1 을 1초마다 출력하는 ubuntu1 이라는 이름의 container 한개와,  
hello2 를 1초마다 출력하는 ubuntu2 라는 이름의 container 한개, 총 2개의 container를 Pod가 가지도록 구성해보았다.  
위와 같은 방식으로 원하는 갯수만큼의 container 배포가 가능하다.  

위의 코드를 `multiple-containers.yaml` 로 저장하고, `kubectl apply` 명령어를 통해 Pod를 배포해보자.  

```shell
$ kubectl apply -f multiple-containers.yaml
pod/multiple-containers-pod created
```

Pod가 만들어졌으니 아래와 같이 확인을 해보자.  

```shell
$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
multiple-containers-pod   2/2     Running   0          20s
```

`multiple-containers-pod` 이름으로 Pod가 잘 생성된 것을 확인할 수 있으며, 내부의 2개의 컨테이너 모두 정상적으로 동작하고 있음을 `READY` 의 `2/2` 를 통해 확인할 수 있다.  

그렇다면 이번엔 각각의 container의 log를 확인해보자.  

```shell
$ kubectl logs multiple-containers-pod
error: a container name must be specified for pod multiple-containers-pod, choose one of: [ubuntu1 ubuntu2]
```

앗?? container가 하나인 Pod에서 잘 작동하던 log 를 확인하는 명령어가 이번에는 에러가 발생한다.  
에러 메시지를 확인해보면 container 이름을 명시하라고 나온다.  
Container가 1개인 경우에는 Pod의 이름까지만 명령어에 입력해주면 자동으로 1개밖에 없는 container를 선택하여 log를 보여줬지만, 2개 이상인 경우에는 사용자가 직접 log를 보길 원하는 container 이름을 입력해줘야 한다.  

아래와 같이 -c 옵션을 통해 container 이름을 명시하여 각각의 container의 log를 확인해보자.  

```shell
# ubuntu1 container의 log 확인
$ kubectl logs multiple-containers-pod -c ubuntu1
hello1
hello1
...
```

```shell
# ubuntu2 container의 log 확인
$ kubectl logs multiple-containers-pod -c ubuntu2
hello2
hello2
...
```

마찬가지로 아래의 `kubectl delete` 명령어를 통해 Pod 삭제가 가능하다.  

```shell
# Pod의 이름으로 삭제 가능
$ kubectl delete pods multiple-containers-pod
```

## Pod의 특징

Pod의 기본적인 생성방법에 대해서 살펴봤으니 이번엔 중요한 특징 2가지에 대해서 알아보자.  

### Volume 공유

Kubernetes 에서는 [Volume](https://kubernetes.io/docs/concepts/storage/volumes/){:target="_blank"} 을 통해 다양한 type 의 저장공간을 지원해 주고 있는데, Pod 내의 있는 container 들은 이 Volume 을 공유하여 사용하는 것이 가능하다.  
이것 또한 예제를 통해 좀더 자세히 이해해보자.  

<script src="https://gist.github.com/gentledev10/f8d135111052d7664fd844555fa455f6.js"></script>

먼저 volume을 사용하는 부분에 대해 간단히 살펴보면,  
`spec` 내의 `volumes`를 통해 `test-volume`이라는 이름을 가진 `volume`을 하나 만들었고,  
각각의 `container` 내부에 `volumeMounts`를 통해 각각의 container 별로 원하는 위치에 volume 을 mount 시켜주었다.  
이제 각각의 container가 하는일을 살펴보면,  

- `ubuntu1 container` - volume이 mount된 `/ubuntu1` 위치에 test.log 파일에 5초마다 hello1 문구를 써준다.  
- `ubuntu2 container` - volume이 mount된 `/ubuntu2` 위치에 test.log 파일을 tail 명령어로 계속 출력한다.  

각각의 container가 mount한 위치는 `/ubuntu1`, `/ubuntu2`로 다르지만 같은 `test-volume` 을 사용하였으므로 동일한 volume이 공유가 되고 있다.  
결국 ubuntu2 에서 출력하고 있는 test.log 파일은 ubuntu1 에서 쓰여지고 있는 test.log 파일과 동일한 것이다.  

ubuntu2의 log를 확인해 보면 더욱 확실해진다.  
이번에는 log 확인시에 -f 옵션으로 계속해서 지켜봐보자.  
5초에 한번씩 ubuntu1에서 파일에 쓰고 있는 내용이 ubuntu2 log에 나타남을 확인할 수 있다.  

```shell
$ kubectl logs volume-share-pod -c ubuntu2 -f
hello1
hello1
...
```

이와 같이 Pod 내부의 container는 volume 을 공유할 수 있음을 확인해 보았다.  

### Network 공유

또 한가지 특징은 Pod 내부의 container들은 network를 공유한다는 점이다.  
간단히 말해 서로 `localhost`로 통신이 가능하다는 뜻이다.  
이것 또한 예제를 통해 살펴보자.  

<script src="https://gist.github.com/gentledev10/1693ab8e288946c8036d8be4a86b8374.js"></script>

편의를 위해 예제에서 사용하는 2개의 docker image를 만들어 배포해놓았다.  

- `gentledev10/spring-boot-docker` - "8080 container" 를 return 하는 API를 제공하며 8080 port를 사용하는 spring boot docker image.
- `gentledev10/ubuntu-curl` - API 호출 테스트를 위해 curl이 포함된 ubuntu docker image.  

예제 yaml 파일을 살펴보면,  
위에서 설명한 spring boot, ubuntu 이미지를 pod 내의 각각의 container로 배포하였으며,  
ubuntu container 에서는 5초마다 localhost:8080 을 호출하여 결과값을 print 해주고 있다.  
ubuntu container 의 log 를 살펴보자.  

```shell
$ kubectl logs network-share-pod -c ubuntu-curl -f
8080 container
8080 container
...
```

위와 같이 같은 Pod 내의 8080 port 로 배포된 spring-boot container의 API 가 localhost 로 호출되어 결과값이 return 되고 있음을 확인할 수 있다.  
즉 network를 공유하고 있기 때문에 localhost 통신이 가능하다는 걸 확인할 수 있다.  
만약 다른 Pod 에 배포되어 있다면 network를 공유하지 않기때문에 각각 Pod에 할당된 IP 주소를 통해 호출하여야 한다.  

## Pod 구성

예제를 통해 Pod에 대해 감을 익혔으니 공식 페이지에 있는 그림으로 Pod의 구성에 대해 한번 살펴보자.  

![Pod]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes/pod/pod.jpg)
출처: https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/

- `Pod1` - 10.10.10.1 IP 주소를 가지고 1개의 container를 가진 Pod.
- `Pod2` - 10.10.10.2 IP 주소를 가지고 1개의 container와 1개의 volume을 가진 Pod.
- `Pod3` - 10.10.10.3 IP 주소를 가지고 2개의 container와 1개의 volume을 가진 Pod. 2개의 container들은 1개의 volume 공유가 가능하다. 또한 network 공유도 가능하여 localhost로 통신이 가능하다.  
- `Pod4` - 10.10.10.4 IP 주소를 가지고 3개의 container와 2개의 volume을 가진 Pod. 3개의 container들은 2개의 volume 모두 공유가 가능하다. 또한 network 공유도 가능하여 localshot로 통신이 가능하다.  

마지막으로 아래와 같이 정리해 볼 수 있겠다.  

Pod란

- 각각 고유의 IP주소를 부여받는다  
- 한개 또는 여러개의 container를 가질 수 있다  
- 동일한 Pod내의 container들은 volume을 공유할 수 있다  
- 동일한 Pod내의 container들은 network를 공유하여 localhost 통신이 가능하다  

<br>
## 마무리

Kubernetes의 가장 기본인 Pod에 대해서 예제와 함께 살펴보았다.  
Pod는 application을 배포하고 관리하는데 사용되므로 kubernetes cluster 운영시에도 가장 많이 확인하는 object가 아닌가 생각이 된다.  
따라서 위의 나온 예제 말고도 본인이 원하는 다양한 환경을 만들어서 배포해보는 연습을 한다면 Pod와 친해지는데 더욱 많은 도움이 될 것이다!  

예제에서 사용한 코드는 모두 아래에서 확인 가능하다.  
<https://github.com/gentledev10/k8s_examples/tree/main/pod>{:target="_blank"}

<br>
<br>
**[참고]**  

<https://kubernetes.io/docs/concepts/workloads/pods/>{:target="_blank"}
<https://bcho.tistory.com/1256?category=731548>{:target="_blank"}
