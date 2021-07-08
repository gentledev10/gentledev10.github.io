---
title: "CKA, CKAD 시험 후기"
excerpt: "CKA, CKAD 시험과 관련 팁들을 공유한다"
categories:
  - Kubernetes
tags:
  - Kubernetes
  - Devops
---

# CKA, CKAD 정보

CKA (Certified Kubernetes Administrator), CKAD (Certified Kubernetes Application Developer) 모두 Linux Foundation에서 주관하는 Kubernetes 자격증으로 각각 아래의 web page 에서 신청이 가능하다.  

- CKA - <https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/>{:target="_blank"}
- CKAD - <https://training.linuxfoundation.org/certification/certified-kubernetes-application-developer-ckad/>{:target="_blank"}

위의 신청 페이지에서 정보를 확인할 수 있지만, CKA, CKAD 모두 아래와 같이 운영된다.  

- **응시료**: $375 (2021년 7월부터 $300에서 올랐다 ㅠㅠ)  
- **시험시간**: 2시간  
- **자격증 유효기간**: 3년  
- **시험가능 기간**: 1년 (시험을 결제하고 1년내로 자유롭게 원하는 날짜에 시험을 볼 수 있다.)  
- **재시험 가능**: 첫번째 시험에서 실패하면 1년 내로 무료로 한번 더 시험을 응시할 수 있는 기회가 주어진다.  
- **외부 사이트 접속 불가**: 시험중엔 공식 사이트인 <https://kubernetes.io>{:target="_blank"} 외 다른 사이트는 접속이 불가능하다.  
- **문제 유형**: 문제가 나오고 정답을 찾는게 아닌 문제의 요구사항을 직접 구현하는 방식이다. 또는 문제상황을 직접 해결하는 문제도 출제된다.  

CKA, CKAD 의 차이점이 있다면 시험을 보는 내용이다.  
CKA 는 Kubernetes cluster 운영 및 관리, resource 들의 사용법에 대한 문제가 나오고,  
CKAD 는 Kubernetes resource 사용법에 중점을 맞춘 문제들이 출제된다.  

# 준비과정

**[강의]**  
시험 준비시작과 동시에 아래의 Udemy 강의를 구매하여 공부했다.  

- CKA - <https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/>{:target="_blank"}
- CKAD - <https://www.udemy.com/course/certified-kubernetes-application-developer/>{:target="_blank"}

Kubernetes 자격증 공부를 하는 사람이라면 대부분 활용하는 강의가 아닐까 생각될 정도로 많은 사람들이 수강하는 인기 강의이다!  
강의 자체도 좋지만 매 강의후에 KodeKloud 라는 곳에서 제공하는 연습문제를 online 으로 풀 수 있는 점이 굉장히 매력적이다.  
또한 Mock test 도 제공해서 실제 시험을 보듯이 (물론 환경은 다르지만) 연습을 해볼 수 있는 환경도 제공한다.  
강의의 정가는 10만원이 넘지만 항상 할인을 해주고있어 1~3만원 정도에 강의를 구매할 수 있으니 가격면에서도 큰 부담이 되지 않는다.  
정말 이 강의 하나만 제대로 마스터해도 무조건 자격증을 딸 수 있다는 생각이 들 정도로 추천한다!  

**[Exam Simulator]**  
시험에 등록하게 되면 Exam Simulator 라고 하는 일종의 Mock test 를 제공해준다. (2021.06.02 부터 제공)  
Udemy 강의를 모두 듣고 시험보기 며칠전엔 Exam Simulator 에 접속해서 제공하는 문제들을 풀어봤다.  
문제에 대한 해설 및 해답도 상세하게 제공해주고 있어 연습하기에 도움이 된다.  
참고로 정해진 시간동안 시험보듯이 푸는 방식인데 실제 시험보다는 난이도가 더 높으므로 다 풀지 못했다고 좌절은 하지 않아도 된다!  

# 팁

**[북마크 활용]**  
Udemy 강의를 듣고 연습문제를 풀때 참고한 Kubernetes 공식 페이지 중 필요한 페이지는 북마크를 해서 시험볼 때 바로바로 접근 할 수 있도록 준비했다.  
각각 Resource의 yaml 형식을 모두 외우고 있을 순 없으므로 yaml 형식이 나온 페이지를 주로 북마크 했다.  
특히 <https://kubernetes.io/search/>{:target="_blank"} 페이지에서 모르는 모든걸 검색이 가능하므로 반드시 북마크해서 활용하길 바란다!  

**[kubectl 명령어 사용]**  
Resource 생성시에 yaml 파일로 만드는 것보다 kubectl 명령어를 사용해서 바로 생성해주는게 시간면에서 굉장히 유리하다.  
많이 사용하는 명령어에 대해선 자주 사용해보고 익힌후에 yaml 파일을 생성하기보단 명령어로 처리하는게 시험볼때 굉장히 도움이 된다.  

**[시험 도중]**  
*Cluster 변경*  
시험 환경에서는 여러개의 cluster 가 제공되고 모든 문제는 주어진 cluster 에서 구현을 해야한다.  
모든 문제의 첫번째 줄에 사용해야 할 cluster 와 사용 명령어가 제공되므로 문제를 풀기 전 반드시 해당 cluster 로 변경하여 구현하기 바란다.  
누구나 쉽게 겪을 수 있는 실수이므로 ~~(저도 실수를 했다 다행히 발견..)~~ 반드시 조심하자!  

*쉬운 문제부터*  
문제별로 점수비중이 나오는데 2~4% 로 표시된 문제들이 쉬운 문제들이다.  
이런 문제들은 대부분 5분내로 풀 수 있는 문제들로 먼저 푸는걸 추천한다.  
어려운 문제들은 체크를 해놓을 수가 있는데 바로바로 체크하고 넘어간 뒤 쉬운문제부터 해결하면 시간의 압박으로부터 조금이나마 자유로울 수 있다.  

*제공되는 note 활용*  
시험 도중에 다른 Text editor 는 사용할 수 없지만 시험 환경에서 간단히 사용할 수 있는 note 를 제공해준다.  
문제를 풀때 잠시 저장해 놔야할 text 들을 노트에 적어두고 복사해가며 문제를 풀었는데 도움이 됐다.  
tmux 를 사용해서 terminal 을 분할해서 시험을 보는 경우도 많은 것 같은데, 시험 도중 그정도로 tmux 의 필요성을 못느껴서 제공되는 note 를 활용했는데 이것만으로도 시험보는데는 문제가 없을 것 같다.  

**[CKA, CKAD]**  
CKA, CKAD 두개의 시험을 모두 봤는데 중복되는 부분이 많아서 CKA 시험 후 CKAD 시험을 준비할때는 추가적인 부분만 조금 공부하고 시험을 볼 수 있었다.  
혹시 CKA 자격증을 따신 분이라면 CKAD 도 큰 무리없이 통과 할 수 있을거라 생각되므로 기회가 된다면 시도해 보는것도 좋을것 같다.  

# 후기

회사에서 Kubernetes 를 2년정도 사용해오다가 어느순간 자격증이라도 하나 따놓을까? 라는 생각으로 찾아보니 시험도 주어진 상황을 구현하는 방식이라 맘에 들어서 준비를 시작하게 됐다.  
사실 시험 비용에 대한 부담이 있었지만.. ~~(너무 비싸다)~~ 지나고 보니 덕분에 Kubernetes 에 대해 조금 더 공부하고 알 수 있게 되서 도움이 많이 됐다.  
실제로 회사에서 Kubernetes 사용 시에 공부하며 알게된 다양한 방법들을 적용해보며 공부하길 잘했다는 생각도 종종 든다.  
혹시 Kubernetes 를 사용중이라면 누구나 충분히 자격증을 딸 수 있다! 도전을 적극 추천한다!  

![CKA]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes/cka-ckad/cka.png)  
![CKAD]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes/cka-ckad/ckad.png)  
