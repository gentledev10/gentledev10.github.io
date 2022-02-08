---
title: "Kubernetes ReplicaSet"
excerpt: "Kubernetes ReplicaSet에 대해 살펴보자"
categories:
  - Kubernetes
tags:
  - Kubernetes
  - Devops
---

ReplicaSet(레플리카셋)은 관리하는 Pod를 원하는 갯수만큼 유지시켜주는 역할을 하는 controller 이다.  

## 동작방식

ReplicaSet은 지속적으로 상태를 체크하여 원하는 갯수만큼의 Pod를 유지하고자 한다.  
따라서 동작방식은 굉장히 간단하며 결국 아래와 같은 두가지 상황에 따라 동작방식이 정해진다.  

- **Pod 갯수가 부족할때** - ReplicaSet이 관리하고자 하는 Pod의 갯수보다 실제 동작중인 Pod의 갯수가 부족하다면 부족한 만큼 추가적인 Pod를 생성하여 갯수를 맞춘다.  
예) ReplicaSet이 3개의 Pod를 관리하는 도중 어떠한 이유로 인해 Pod의 갯수가 2개로 줄었다면 ReplicaSet이 추가적인 Pod 한개를 생성해 3개로 유지한다.  
- **Pod 갯수가 초과할때** - ReplicaSet이 관리하고자 하는 Pod의 갯수를 실제 동작중인 Pod의 갯수가 초과한다면 초과된 갯수만큼 Pod를 종료하여 없앤다.  
예) ReplicaSet이 3개의 Pod를 관리하는 도중 어떠한 이유로 Pod의 갯수가 4개로 늘었다면 ReplicaSet이 Pod 한개를 제거해 3개로 유지한다.  

## ReplicaSet 생성

> 아래의 예제들을 수행시에는 ReplicaSet 이름이 동일하므로 반드시 이전에 생성된 ReplicaSet을 다음과 같이 삭제하고 새로 생성해야한다.
> ```shell
> $ kubectl delete replicasets <ReplicaSet 이름>
> ```

<script src="https://gist.github.com/gentledev10/3d2e2673eebd0630d55a4859e49bdba7.js"></script>

위의 예제 `yaml` 파일을 통해 작성법에 대해 알아보고 ReplicaSet를 직접 생성해보자.  

`kind`는 당연하게도 ReplicaSet 생성을 위해 `ReplicaSet` 을 선언했고 `metadata`에는 ReplicaSet의 이름과 label을 설정해 주었다.  
가장 중요한 부분인 `spec`을 확인해보자.  

- `replicas` - ReplicaSet이 유지하고자 하는 Pod의 갯수이다.  
ReplicaSet은 지속적으로 Pod의 갯수를 확인하여 정상적으로 동작하고 있는 Pod의 갯수를 `replicas`에 정의한 갯수와 동일하게 맞춰준다.  
예제에서는 3개의 Pod를 관리하도록 설정해 주었다.  
- `selector` - ReplicaSet이 관리하고자 하는 Pod를 선택하기 위한 조건을 정의하는 필드이다.  
예제에서는 `matchLabels`를 사용하여 `app: ubuntu` 라는 label 을 가진 Pod를 선택하여 ReplicaSet이 관리하도록 조건을 정의하였다.  
이와 같이 ReplicaSet은 아무 Pod나 선택해서 관리하는 것이 아니라, 관리하고자 하는 Pod를 `selector` 필드를 통해 선택하여 관리하게 된다.  
`selector`에는 다양한 조건들의 추가가 가능한데 방법에 대해서는 [Label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors){:target="_blank"} 에서 확인 가능하다.  
- `template` - ReplicaSet을 통해 생성되고 관리되고자 하는 Pod에 대해 정의하는 필드이다. [Pod를 정의할때]({% post_url /kubernetes/2021-12-20-what-is-the-pod %}){:target="_blank"}와 마찬가지로 `metadata`와 `spec`에 대해 정의해주면 된다.  
ReplicaSet은 관리하고 있는 Pod의 갯수가 유지하고자 하는 Pod의 갯수보다 부족할때, 여기에 정의된 Pod를 생성하여 Pod의 갯수를 유지시킨다. 따라서 Pod의 `metadata`에 정의된 `label`은 반드시 ReplicaSet의 `selector`와 match가 되어야한다.  

더욱 자세한 작성방법을 확인하고 싶다면 [ReplicaSet refenrece](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/replica-set-v1/#ReplicaSet){:target="_blank"}를 통해 확인 가능하다.  
YAML 파일의 작성방법에 대해 알아보았으니 이제 위의 YAML 파일을 `replicaset.yaml` 파일로 저장후 아래 2가지 상황에 따라 직접 배포해보자.  

### 매칭되는 Pod가 존재하지 않는 경우

먼저 매칭되는 Pod가 없는 경우에 ReplicaSet을 배포하는 경우부터 살펴보자.  
아래의 명령어로 배포를 시작한다.  

```shell
$ kubectl apply -f replicaset.yaml
```

배포가 시작되었으니 ReplicaSet과 Pod의 상태를 확인해보자.  

```shell
# ReplicaSet 생성 확인
# kubectl get rs 명령어도 동일하게 사용 가능
$ kubectl get replicasets
NAME     DESIRED   CURRENT   READY   AGE
ubuntu   3         3         0       2s

# Pod 생성 확인
$ kubectl get pods
ubuntu-9vgv2   0/1     ContainerCreating   0          4s
ubuntu-j9kvl   0/1     ContainerCreating   0          4s
ubuntu-xcww2   0/1     ContainerCreating   0          4s
```

ReplicaSet이 생성되면 3개의 Pod를 관리하고자 하는데 현재 매칭되는 Pod가 없으므로 우리가 ReplicaSet에 정의했던 Pod template을 사용해서 3개의 Pod가 생성되고 있는걸 확인할 수 있다.  
각각의 Pod는 동일 namespace에서 동일한 이름을 가질 수 없으므로 5자리의 random string이 뒤에 붙는다.  
ReplicaSet의 상태는 `DESIRED`가 `spec`에서 `replicas`로 정의한 3개, `CURRENT`는 Pod 3개가 생성되어 있으므로 3개, `READY`는 아직 Pod가 생성 중이므로 0개로 되어 있는걸 확인할 수 있다.  
Pod가 running 상태가 된 후 다시 ReplicaSet의 상태를 확인해보면 아래와 같이 정상적으로 동작되고 있음을 확인할 수 있다.  

```shell
$ kubectl get replicasets
NAME     DESIRED   CURRENT   READY   AGE
ubuntu   3         3         3       20s
```

그림으로 살펴보면 아래와 같이 ReplicaSet이 3개의 새롭게 생성된 Pod를 관리하고 있는 상태가 된다.  

![ReplicaSet manages new 3 pods]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes/replicaset/replicaset-3-new-pods.png)

### 매칭되는 Pod가 이미 존재하는 경우

이번엔 `selector`에 매칭되는 Pod가 이미 존재하는 경우 ReplicaSet의 동작에 대해 살펴보자.  
동작 확인을 위해서는 미리 배포가 된 Pod가 필요하므로 먼저 아래의 예제코드를 통해 매칭되는 Pod를 배포한다.  

<script src="https://gist.github.com/gentledev10/c2d9a1461130097d2246685cf2b4b808.js"></script>

Pod 배포가 완료되면 ReplicaSet을 배포하여 동작을 살펴보자.  
ReplicaSet은 위에서 사용한 파일을 그대로 사용하여 배포한다.  

```shell
# ReplicaSet 배포
$ kubectl apply -f replicaset.yaml

# ReplicaSet 상태 확인
$ kubectl get replicasets
NAME     DESIRED   CURRENT   READY   AGE
ubuntu   3         3         1       2s

# Pod 상태 확인
$ kubectl get pods
manual-pod     1/1     Running             0          7m53s
ubuntu-br945   0/1     ContainerCreating   0          4s
ubuntu-c4z84   0/1     ContainerCreating   0          4s
```

우리가 위에서 살펴봤듯이 RepliaSet은 `selector`와 매칭되는 Pod를 관리하기 때문에, 이미 매칭되는 Pod가 존재한다면 해당 Pod를 바로 관리하기 시작한다.  
따라서 아래 그림처럼 이미 매칭이 되는 `manual-pod` Pod 1개와 추가적으로 생성된 2개의 Pod를 관리하게 된다.  

![ReplicaSet manages 1 ond pod and new 2 pods]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes/replicaset/replicaset-2-new-pods.png)

단 이런경우 의도하지 않게 이미 존재하는 Pod가 관리되는 경우가 생길 수 있으므로 주의해야 한다.  

## Pod를 어떻게 관리하나?

ReplicaSet의 생성을 통해 동작에 대해 살펴봤으니, 이번엔 ReplicaSet이 어떤 방식으로 Pod를 관리하고 있는지에 대해서 살펴보자.  

결론부터 얘기하자면 ReplicaSet은 Pod의 `metadata`에 있는 `ownerReferences` 필드를 통해 서로 연결된다.  
좀더 자세히는 Pod의 `ownerReferences` 필드가 없거나, 필드가 있지만 controller가 아닌 경우 ReplicaSet의 `selector`와 매칭된다면 서로 연결되어 관리를 시작한다.  

위의 예제에서 `replicaset.yaml` 파일을 통해 배포된 ReplicaSet과 Pod를 통해 직접 확인해보자.  

```shell
# ReplicaSet 배포
$ kubectl apply -f replicaset.yaml

# ReplicaSet과 3개의 Pod가 배포된 상태
$ kubectl get replicasets
NAME     DESIRED   CURRENT   READY   AGE
ubuntu   3         3         3       23h

$ kubectl get pods
ubuntu-br945   1/1     Running   0          23h
ubuntu-c4z84   1/1     Running   0          23h
ubuntu-jq6n6   1/1     Running   0          23h
```

배포된 Pod 중 하나를 선택해 정보를 확인해보자.  

```shell
# -o yaml 옵션을 통해 Pod의 정보를 yaml 형식으로 보여준다.
# 본인의 환경에 배포된 Pod 명으로 변경 필요!
$ kubectl get pods ubuntu-jq6n6 -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2022-02-07T16:53:08Z"
  generateName: ubuntu-
  labels:
    app: ubuntu
    env: test
  name: ubuntu-jq6n6
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: ubuntu
    uid: e89f0fdd-fdec-476e-9877-b21dd108eaca
...
...
...
```

위와 같이 `metadata`에 `ownerReferences` 필드가 추가되어 있는 것을 확인할 수 있다.  
내용을 살펴보면 owner는 ubuntu 라는 이름의 ReplicaSet이며 controller 임을(`controller: true`) 알 수 있다.  
이 정보를 통해 ubuntu ReplicaSet은 ubuntu-jq6n6 Pod와 연결되어 관리를 하고 있는 것이다.  
만약 새로운 Pod가 배포됐는데 `ownerRefenrences` 필드가 없거나, 필드가 존재하지만 `controller: false`(controller가 아닌 경우)에 `selector`와 매칭된다면 위와 같이 `ownerReferences` 필드가 추가되고 ReplicaSet의 정보가 입력되어 서로 연결이 되는 것이다.  

## 주의사항

<script src="https://gist.github.com/gentledev10/8451b6b1974bf97bf3028ec4aaaac1fb.js"></script>

위의 예제를 `wrong-replicaset.yaml` 파일로 저장하고 배포해보자.  
아래와 같이 정상적으로 배포되지 않고 error 메시지를 출력한다.  

```shell
$ kubectl apply -f wrong-replicaset.yaml
The ReplicaSet "ubuntu" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"app":"not-match-with-selector", "env":"test"}: `selector` does not match template `labels`
```

확인해보면 Pod template의 정의된 Pod의 labels 중 ReplicaSet의 `selector`와 매칭되는 label이 없다는 내용이다.  
생각해보면 ReplicaSet은 Pod의 갯수가 부족할때 여기에 정의된 Pod를 생성하여 관리하므로 원하는 갯수 충족을 위해서 당연히 `selector`와 매칭이 되서 Pod 생성시 해당 ReplicaSet과 연결되어 관리가 되야한다.  
그렇지 않다면 ReplicaSet은 Pod의 갯수를 충족하기 위해 template에 정의된 매칭되지 않는 Pod를 무한대로 생성하는 문제가 생길 것이다.  
다행히 이런 문제가 발생하지 않도록 Kubernetes에서는 배포시 위와 같은 error 메시지를 출력하여 개발자에게 알려준다.  
ReplicaSet 배포 시 위와같은 error를 만나게 된다면 당황하지 말고 Pod를 `selector`와 매칭시켜주자.  

<br>
## 마무리

ReplicaSet의 동작 방식, 원리, 생성에 대해 확인해 보았다.  
~~마무리를 하는 시점에 당황스러운 얘기지만..~~ 사실 동일한 역할을 하는 controller로는 ReplicaSet보다 high level인 Deployment를 사용하는 것이 바람직하며 Kubernetes에서도 공식적으로 추천하는 방식이다.  
실제 서비스를 운영하는 상황에서는 무중단 배포도 중요 고려사항 중 하나인데, ReplicaSet에서는 어려운 배포 전략을 Deployment는 제공해주고 그외 rollback 같은 기능도 가능하기 때문이다.  
그럼에도 우리가 ReplicaSet을 살펴본 이유는 Deployment가 ReplicaSet을 생성하고 관리하는 동작을 하기때문에 결국은 ReplicaSet을 잘 알아야 Deployment도 잘 사용할 수 있기 때문이다!  

<br>
<br>
**[참고]**  

<https://kubernetes.io/docs/concepts/workloads/controllers/replicaset>{:target="_blank"}
<https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/replica-set-v1>{:target="_blank"}
