---
title: "Terraform 이란?"
excerpt: "Terraform에 대해 알아보자"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# IaC

IaC 란 Infrastructure as Code 를 뜻하는 용어이다.  
쉽게 예를 들자면, 우리가 만든 Application 을 돌리기 위한 infrastructure 를 code 를 통해 구축한다라고 생각하면 좋을 것 같다.

더 구체적인 예를 들어 보겠다.  
만약 Spring boot app 을 만들었고, 이 app을 AWS의 EC2에서 실행시키기 위해 AWS infrastructure 를 구축해야 한다고 하자.  
IaC 방식을 사용하지 않는다면, AWS console 에 접속해서 EC2 서비스를 선택하고 Next, Next 를 눌러가며 EC2 인스턴스를 만들고 app 을 실행시켜야 한다.  
(물론 AWS CLI 를 활용하는 방법도 있다.)  
하지만 이 모든 절차를 code 로 구현해 구축하는 것이 바로 IaC 이다.

# Terraform

자 그럼 이제 Terraform에 대해 알아보자.  
Terraform 이란 code 를 통해 infrastructure 를 구성하고 관리할 수 있도록 해주는 open source tool 이다.  
기본적으로 HashiCorp Configuration Language (HCL) 을 사용하여 code 를 작성할 수 있으며 json 으로도 가능하다.  
(심지어 최근에는 python, typescript 로도 가능하다! [참고 site](https://www.hashicorp.com/blog/cdk-for-terraform-enabling-python-and-typescript-support))  
작성된 code를 통해 infrastructure resource 들을 생성, 변경, 삭제가 가능하다.

그렇다면 Terraform 을 사용해서 얻게되는 장점은 무엇일까?  
실무에서 사용하며 느낀점을 토대로 적어보겠다.

## 장점

- Open source 이고 IaC 를 위한 도구로 많은 사람들이 이용하고 있기 때문에, 문제가 생겼을때 쉽게 검색해서 해결법을 찾을 수 있고, 다양한 이슈들에 대해서도 활발이 수정 및 업데이트가 되고 있다.
- Terraform code 를 git 으로 관리한다면 기존에는 하기가 힘들었던 infrastructre 구성 history 를 관리할 수 있다.
- Code 로 관리가 되기 때문에 특정 resource 를 추가하기 위해 적용 전 다른 사람의 리뷰를 받을 수 있다.
- 기존 Code 를 재사용 함으로써 추가적으로 동일한 resource 를 만들때 쉽고 빠르게 적용이 가능하다.

Console 에서 launch wizard 같은 걸 통해 원하는 resource 를 재빨리 만드는거에 익숙한 사람이라면, Terraform 을 사용해서 동일한 resource 를 만드는데는 분명 시간이 더 필요할 것이다. (문서를 봐가며 원하는 configuration 들을 추가해 줘야 하기 때문에)  
하지만 분명 Terraform 을 통해 infrastructure 를 구성한다면, 위에 적은 것처럼 기존 console 에서 구성하던 방식과는 다르게 더 많은 장점을 가져갈 수 있고 더 안정적인 infrastructure 를 구성할 수 있을 것이다.

다음 post 부터는 Terraform 을 통해 infrastructure 를 구성하는 방법에 대해 설명해보도록 하겠다.

