---
title: "Kubernetes Deployment"
excerpt: "Kubernetes Deployment에 대해 살펴보자"
categories:
  - Kubernetes
tags:
  - Kubernetes
  - Devops
---

Deployment(디플로이먼트)는 ReplicaSet을 관리하는 controller 이다.  
또한 ReplicaSet에서는 지원하지 않았던 업데이트에 관한 설정까지도 지원하여 더욱 안정적으로 container들의 운영이 가능하다.  
직접 Deployment를 생성해보며 어떻게 사용하는지 살펴보자.  
(혹시 ReplicaSet에 대해 아직 모른다면 [이전 글]({% post_url /kubernetes/2022-02-08-kubernetes-replicaset %}){:target="_blank"}을 확인바란다.)  

## Deployment 생성 및 동작방식

<script src="https://gist.github.com/gentledev10/9a8d6ed1ee632951bf7bc0819d935a79.js"></script>

위의 예제 `yaml` 파일을 보면 한가지 발견할 수 있는 점이 있다.  
그렇다! ReplicaSet을 생성할때와 `kind`가 바뀐 것 빼고는 완전히 동일하다.  
Deployment가 결국 ReplicaSet을 관리하는 controller이기 때문인데, 하나씩 살펴보자.  

`kind`는 당연하게도 `Deployment`로 선언해주었고, `metadata`에는 Deployment의 이름과 label을 설정해 주었다.  
`spec`에 대해 살펴보자.  

- `replicas` - 유지하고자 하는 Pod의 갯수이다.  
여기에 명시한 갯수만큼 정상적으로 동작하는 Pod의 갯수를 유지시킨다. (실행하려는 Pod가 정상적이지 않는 경우에 불가피하게 정상동작하는 Pod의 갯수를 유지하지 못하는 경우가 발생하기도 한다.)
- `selector` - 관리하고자 하는 Pod를 선택하기 위한 조건을 정의하는 필드이다.  
예제에서는 `matchLabels`를 사용하여 `app: nginx` 라는 label 을 가진 Pod를 선택하여 관리하도록 조건을 정의하였다.  
`selector`에는 다양한 조건들의 추가가 가능한데 방법에 대해서는 [Label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors){:target="_blank"} 에서 확인 가능하다.  
- `template` - 관리되고자 하는 Pod에 대해 정의하는 필드이다. [Pod를 정의할때]({% post_url /kubernetes/2021-12-20-what-is-the-pod %}){:target="_blank"}와 마찬가지로 `metadata`와 `spec`에 대해 정의해주면 된다.  
관리하고 있는 Pod의 갯수가 유지하고자 하는 Pod의 갯수보다 부족할때, 여기에 정의된 Pod를 생성하여 Pod의 갯수를 유지시킨다. 따라서 Pod의 `metadata`에 정의된 `label`은 반드시 Deployment의 `selector`와 match가 되어야한다.  
예제에서는 nginx 1.20.2 버전의 container를 배포하고자 한다.  

더욱 자세한 작성방법을 확인하고 싶다면 [Deployment refenrece](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/){:target="_blank"}를 통해 확인 가능하다.  

이제 위의 YAML 파일을 `deployment.yaml` 파일로 저장후 직접 배포해보자.  

```shell
$ kubectl apply -f deployment.yaml
```

배포가 시작되었으니 Deployment, ReplicaSet, Pod의 상태에 대해 차례대로 살펴보자.  

먼저 Deployment 상태에 대해 확인해보면 총 3개의 Pod가 정상적으로 배포되어 있는걸 확인해 볼 수 있다.  

```shell
# Deployment
$ kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           39s
```

- `READY` - *3/3* 은 우리가 요청한 3개의 replicas중 3개가 정상 동작중임을 나타낸다.  
만약 3개중 2개만 정상동작한다면 *2/3* 으로 표시될 것이다.  
- `UP-TO-DATE` - 선언된 replicas의 갯수를 위해 update 된 replicas의 갯수를 나타낸다. (즉 최신의 PodTemplate를 사용하여 배포되었음을 의미)  
예제에서는 3개의 replicas를 선언하였고, 이를 위해 3개의 Pod가 새롭게 생성되었으므로(update 되었으므로) 3으로 표시된다.  
- `AVAILABE` - 사용 가능한 replicas의 갯수가 표시된다.  

이번엔 Deployment에 의해 배포되고 관리되고 있는 ReplicaSet에 대해 살펴보자.  

```shell
# ReplicaSet
$ kubectl get replicasets
NAME               DESIRED   CURRENT   READY   AGE
nginx-5777589579   3         3         3       78s
```

ReplicaSet이 정상적으로 3개의 Pod를 관리하고 있는 걸 확인해볼 수 있다.  
아래의 명령어를 통해 조금 더 자세히 살펴보며 추가적으로 2가지에 대해 알아보자.  

```shell
# nginx-5777589579 ReplicaSet에 대해 배포되어 있는 정보를 yaml 형태로 출력
$ kubectl get replicasets nginx-5777589579 -o yaml
# 명령어 결과 중 확인해볼 부분만 남기고 나머진 삭제하였다.
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: nginx
    pod-template-hash: "5777589579"
  name: nginx-5777589579
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: Deployment
    name: nginx
    uid: 502b3059-7163-4d42-b2a3-cc7172473cb7
spec:
  selector:
    matchLabels:
      app: nginx
      pod-template-hash: "5777589579"
  template:
    metadata:
      labels:
        app: nginx
        pod-template-hash: "5777589579"
```

1. **ReplicaSet 이름** - ReplicaSet만을 배포했을 때와는 다르게 Deployment를 통해 생성된 Replicaset의 이름은 다음과 같은 규칙을 가지고 있다  
-> `[Deployment 이름]-[pod-template-hash]`  
Deployment를 통해 배포된 ReplicaSet에는 `spec.template`에 해당하는 Pod template를 hash한 값이 `pod-template-hash` label로 사용되고 이름에도 사용된다.  
*(공식 문서에는 `pod-template-hash` 값을 seed로 사용하여 random string을 생성해 이름으로 사용한다고 하는데, 다양하게 테스트를 해봐도 결국 `pod-template-hash`값 자체를 이름으로 사용하는 것으로 확인되는데 혹시 해당 부분에 대해 오류가 있다면 제보주세요 :D)*  
또한 `selector`에도 추가되어 Pod에도 해당 label이 추가되는걸 확인할 수 있다.  
이 동작은 Deployment가 관리하는 ReplicaSet들이 서로 충돌되지 않도록 하는 중요한 역할을 한다.  
2. **ownerReferences** - ReplicaSet은 `metadata.ownserReferences` 필드를 통해 Deployment에 의해 관리된다. 내부 내용을 살펴보면 이름이 nginx인 Deployment에 의해 관리되고 있음을 확인할 수 있다.  

이번에는 Pod도 잘 동작하고 있는지 살펴보자.  

```shell
# Pod
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-5777589579-6kxxk   1/1     Running   0          2m
nginx-5777589579-8ftjs   1/1     Running   0          2m
nginx-5777589579-gz28b   1/1     Running   0          2m
```

3개의 Pod가 정상적으로 동작하고 있음을 확인할 수 있고, ReplicaSet의 이름에 추가적인 random 문자가 붙어 이름이 생성된걸 확인할 수 있다.  
Pod또한 `onwerReferences` 필드를 통해 ReplicaSet에 의해 관리된다.  

## Update 방식

위에서는 기본적인 Deployment의 생성법에 대해 알아봤다면 이번에는 Deployment가 ReplicaSet을 대체하여 사용되는 가장 큰 이유인 Update 방식에 대해 살펴보자.  
Update가 진행되면 Deployment는 update된 ReplicaSet을 생성하여 새로운 Pod를 만들고, 기존 ReplicaSet에서 관리되던 Pod는 모두 제거한다.  
Update 방식에는 2가지 방식(Recreate, RollingUpdate)을 제공하며 `spec.strategy` 필드에 정의가 가능하다.  
Deployment시에 update 관련하여 아무 설정도 해주지 않았다면 기본적으로는 `RollingUpdate` 방식이 default로 지정된다.  
각각 예제와 함께 알아보겠다.  

### Recreate

먼저 살펴볼 Update 방식은 `Recreate` 방식이다.  
Recreate 방식은 *Deployment가 관리중인 모든 Pod를 없앤후에 새로운 Pod를 배포*하는 방식이다.  
따라서 불가피하게 Application이 모두 중단되는 downtime이 발생하게 된다.  
아래의 실습을 통해 확인해보자.  

**(1)** 먼저 아래의 12-13 라인과 같이 `strategy.type`을 `Recreate`로 지정해주고 yaml 파일을 작성한다.  

<script src="https://gist.github.com/gentledev10/07e577c7c7da9ca7ca3ac8d77e33b897.js"></script>

**(2)** 작성된 파일을 `deployment-recreate.yaml`로 저장 후 아래와 같이 cluster에 배포한다.  

```shell
$ kubectl apply -f deployment-recreate.yaml
```

**(3)** 정상적으로 Deployment가 배포되었다면 update가 진행되는 동안 Deployment의 상태변화 관찰을 위해 `watch` 명령어를 사용한다.  
(Terminal을 분할하거나 여러개를 띄워놓은 상태에서 관찰을하면 편리하다.)  

```shell
$ watch "kubectl get deployments nginx"
```

**(4)** Update를 위해 아래의 명령어로 nginx의 버전을 변경하거나,  

```shell
kubectl set image deployment/nginx nginx=nginx:1.21.6
```

또는 yaml파일의 nginx 버전을 `1.21.6`로 변경하여 저장한 뒤 다시 `kubectl apply`를 통해 배포한다.  

배포직후 Deployment의 상태를 보면 순간적으로 `READY`의 상태가 `0/3` 되며 존재하던 Pod가 모두 종료되고 update된 새로운 Pod가 배포되는걸 확인할 수 있다.  
또한 `kubectl`에서 제공하는 update 상태를 체크하는 명령어도 있으니 배포직후 아래와 같이 확인해보길 바란다.  

```shell
# kubectl rollout status deployment/<Deployment 이름>
$ kubectl rollout status deployment/nginx
Waiting for deployment "nginx" rollout to finish: 0 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 0 out of 3 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 0 of 3 updated replicas are available...
Waiting for deployment "nginx" rollout to finish: 1 of 3 updated replicas are available...
Waiting for deployment "nginx" rollout to finish: 2 of 3 updated replicas are available...
deployment "nginx" successfully rolled out
```

마지막으로 위의 과정을 순서대로 살펴보면 아래와 같은 과정을 거치게 될 것이다.  

![Deployment update recreate]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes/deployment/update-recreate.jpg)

### RollingUpdate

이번엔 Default 전략인 RollingUpdate 방식에 대해서 살펴보자.  
이 방식은 기존 Pod를 일부 제거하고 새롭게 Update된 Pod를 일부 배포하는 과정을 반복하는 방식으로 점진적인 배포과정 때문에 완료까지 시간이 더 걸리겠지만 일부의 Pod는 계속 running 상태로 유지가 되기 때문에 Downtime이 발생하지 않는다.  
RollingUpdate는 아래와 같이 2가지 추가적인 설정이 가능하다.  

- `strategy.rollingUpdate.maxSurge` - Update시 최대 얼마만큼의 Pod를 더 생성할 수 있는지 정할 수 있는 설정이다. int 값(ex: 5) 또는 string 값(ex: 20%)으로 %의 사용이 가능하다.  
만약 `replicas` 갯수가 10개인 상황에서 `maxSurge`를 `5`로 설정했다면 update시 기존 Pod와 새로운 Pod를 최대 15개(10 + 5)까지 유지하며 update가 진행될 것이다.  
`maxSurge`를 `20%`로 설정했다면 update시 기존 Pod와 새로운 Pod를 최대 12개(10 + 2(`replicas` 갯수의 20%))까지 유지하며 update가 진행될 것이다.  
- `strategy.rollingUpdate.maxUnavailable` - Update시 최대 얼마만큼의 Pod가 unavailable 상태여도 되는지 정할 수 있는 설정이다.  int 값(ex: 5) 또는 string 값(ex: 20%)으로 %의 사용이 가능하다.  
만약 `replicas` 갯수가 10개인 상황에서 `maxUnavailable`를 `2`로 설정했다면 update 시 10개의 Pod를 기준으로 2개까지 unavailable 상태가 가능하고 8개는 항상 정상동작 하는 상태가 유지 될 것이다.  
`maxUnavailable`를 `20%`로 설정한 경우에도 동일하게 동작할 것이다.  

결국 RollingUpdate는 위의 2가지 설정에 따라 update가 진행되게 된다.  
직접 실습을 해보며 동작에 대해 살펴보자.  

**(1)** 먼저 아래의 12-13 라인과 같이 `strategy.type`을 `RollingUpdate`로 지정해주고 14-16 라인과 같이 추가적인 설정을 해준 뒤 yaml 파일을 작성한다.  
`replicas`가 4개이고 `maxSurge` 값이 `50%`이므로 Pod는 최대 6개까지 생성 가능하며 `maxUnavailable` 값이 `50%`이므로 Pod 4개를 기준으로 최대 2개까지 unavailable 상태가 가능하다.

<script src="https://gist.github.com/gentledev10/ea9a7c16d3ee19747765c0ab2fffc8ba.js"></script>

**(2)** 작성된 파일을 `deployment-rolling-update.yaml`로 저장 후 아래와 같이 cluster에 배포한다.  

```shell
$ kubectl apply -f deployment-rolling-update.yaml
```

**(3)** 정상적으로 Deployment가 배포되었다면 update가 진행되는 동안 Deployment와 Pod의 상태변화 관찰을 위해 `watch` 명령어를 사용한다.  

```shell
$ watch "kubectl get deployments"
```

```shell
$ watch "kubectl get pods"
```

**(4)** Update를 위해 아래의 명령어로 nginx의 버전을 변경하거나,  

```shell
kubectl set image deployment/nginx nginx=nginx:1.21.6
```

또는 yaml파일의 nginx 버전을 `1.21.6`로 변경하여 저장한 뒤 다시 `kubectl apply`를 통해 배포한다.  

배포직후 `watch` 명령어를 통해 Deployment의 상태를 지켜보면 `maxUnavailable`가 50%, 즉 2개이므로 `READY` 의 상태가 `2/4`까지 되는걸 확인할 수 있다.  
또한 동일하게 `watch` 명령어를 통해 Pod의 상태를 지켜보면 `maxSurge`가 50%, 즉 2개이므로 생성되는 Pod가 6개까지 늘어나는 걸 확인할 수 있다.  

물론 배포직후 `kubectl rollout` 명령어로도 배포상태 확인이 가능하다.  

```shell
# kubectl rollout status deployment/<Deployment 이름>
$ kubectl rollout status deployment/nginx
Waiting for deployment "nginx" rollout to finish: 2 out of 4 new replicas have been updated...
Waiting for deployment "nginx" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 2 of 4 updated replicas are available...
Waiting for deployment "nginx" rollout to finish: 3 of 4 updated replicas are available...
deployment "nginx" successfully rolled out
```

마지막으로 위의 과정을 순서대로 살펴보면 아래와 같은 과정을 거치게 될 것이다.  

![Deployment update recreate]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes/deployment/update-rolling.jpg)

## 롤백

개발을 하다보면 새로운 Application을 배포했다가 문제가 생겨 다시 이전 버전으로 롤백을 했던 경험이 분명히 있을 것이다.  
Deployment 또한 이 롤백 기능을 지원하는데 실습을 통해 살펴보자.  

먼저 위에서 Update 실습을 위해 진행했던 과정을 그대로 수행하여 Deployment가 Update된 Pod를 관리하는 상태로 만들어준다.  
이 상태에서 ReplicaSet을 확인해보면 아래와 같이 이전 버전의 ReplicaSet 1개(Pod 0개), Update 된 현재 버전의 ReplicaSet 1개(Pod 4개), 총 2개가 있을 것이다.  

```shell
$ kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-5777589579   0         0         0       3m34s
nginx-77fcc495b8   4         4         4       2m36s
```

롤백을 하기전에 먼저 배포를 했던 이력에 대해 확인할 수 있는 기능에 대해 살펴보자.  

```shell
# kubectl rollout history deployment/<deployment 이름>
$ kubectl rollout history deployment/nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
2         <none>
1         <none>
```

위의 명령어를 사용하여 현재 2개의 이력이 있음을 확인할 수 있다.  
특정 REVISION의 정보를 확인해보고 싶다면 아래와 같이 `--revision` 옵션을 사용한다.  

```shell
$ kubectl rollout history deployment/nginx --revision=2
deployment.apps/nginx with revision #2
Pod Template:
  Labels:	app=nginx
	pod-template-hash=77fcc495b8
  Containers:
   nginx:
    Image:	nginx:1.21.6
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

현재 REVISION 2가 배포되어 있는 상태이며 다시 이전 버전으로 롤백 하기 위해서는 아래와 같은 방법을 사용하면 된다.  

```shell
$ kubectl rollout undo deployment/nginx
```

만약 특정 REVISION으로 롤백을 하고싶다면 `--to-revision` 옵션을 사용하여 롤백할 버전을 명시해주면 된다.  

```shell
$ kubectl rollout undo deployment/nginx --to-revision=1
```

>실습시에 배포 이력을 보면 `CHANGE-CAUSE`가 `<none>` 으로 표시가 되는데 이건 우리가 Deployment 배포시 해당 annotation을 입력하지 않았기 때문이다.  
>`CHANGE-CAUSE` 이력을 남기고 싶다면 아래처럼 명령어를 통해 `kubernetes.io/change-cause` annotation을 입력하거나,
>```shell
>$ kubectl annotate deployment/nginx kubernetes.io/change-cause="Nginx version 1.20.2"
>```
>`yaml`파일 작성시 `metadata.annotations`에 `kubernetes.io/change-cause` 값을 추가해주면 된다.  
>`kubectl` 명령어 사용시 `--record=true`를 사용하여 `CHANGE-CAUSE`에 명령어 자체를 남기는 방법도 있으나 현재 Deprecated된 상태라 따로 설명하진 않겠다.  

<br>
## 마무리

이번 글을 통해 Deployment의 사용법과 Update 전략, 롤백에 관해 알아보았다.  
다양한 Kubernetes controller 중 가장 많이 쓰일것이라 생각되는데, 그만큼 세부 내용들에 대해 더 정확히 알고 사용한다면 더욱 안정적인 Kubernetes cluster를 운영하는데 분명히 도움이 될 것이라 생각한다.  
특히 RollingUpdate시 각자의 cluster 운영 환경에 맞는 전략을 사용한다면 더욱 안정적이고 효율적인 배포에도 도움이 되니 다양하게 테스트 해보는 것을 추천한다.  

<br>
<br>
**[참고]**  

<https://kubernetes.io/docs/concepts/workloads/controllers/deployment>{:target="_blank"}
