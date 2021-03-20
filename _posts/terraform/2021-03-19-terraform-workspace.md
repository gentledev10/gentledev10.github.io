---
title: "Terraform workspace 사용법"
excerpt: "Terraform workspace 를 통해 여러개의 서버환경을 분리하여 관리하자"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Workspace

일반적으로 서버 개발자들은 목적에 따라 여러개의 환경을 구성하여 서버를 구축한다. (일반적인 예가 `dev`, `staing`, `production` 으로 서버를 각각 구축하여 운영하는 것이다.)  
따라서 Terraform 으로 서버 환경을 구축하기 위해서는 이 부분에 대한 고려가 필요한데, 다양한 방법이 있지만 그 중 하나가 Terraform CLI 에서 제공하는 `workspace`다.  
`workspace`란 단어 그대로 구축하고자 하는 서버 환경에 따라 workspace 를 나누어서 각 서버의 상태를 관리하겠다는 뜻이다.  
즉 3개의 서버 환경이 구성되어 있고 3개의 `workspace` 를 만들어서 각각 infrastructure 를 구성하였다면, 총 3개의 상태파일(.tfstate)이 각각 생성되어 관리가 될 것이다.  
예제와 함께 사용법에 대해 더 쉽게 알아보자.  

## 예제 코드

먼저 `workspace` 테스트를 위해 사용할 코드에 대해 먼저 살펴보자.  
<https://github.com/gentledev10/terraform_examples/tree/main/workspace>{:target="_blank"}

<script src="https://gist.github.com/gentledev10/415f772200e14dcadd952f1bfa3dc712.js"></script>

`workspace` 를 테스트 하는 것이 목적이기 때문에 간단한 코드를 사용했으며, region 과 VPC 이름을 input variables로 전달받아 생성하는 예제이다.  
dev, prod 로 환경을 분리하여 테스트 하기위해 `dev.tfvars`, `prod.tfvars` 로 파일을 분리하여 각각 input variables 를 사용하도록 했다.  

## 사용법

이제 위의 예제 코드와 함께 workspace 사용법에 대해 익혀보자.  
터미널에서 다음과 같이 workspace 의 명령어들을 확인할 수 있다.  

```bash
$ terraform workspace -h
Usage: terraform workspace

  new, list, show, select and delete Terraform workspaces.

Subcommands:
    delete    Delete a workspace
    list      List Workspaces
    new       Create a new workspace
    select    Select a workspace
    show      Show the name of the current workspace
```

먼저 `dev` 환경을 구성하기 위해 `dev` workspace 를 생성해보자.  
아래의 명령어를 통해 `dev` workspace 를 생성과 동시에 선택할 수 있다.  

```bash
$ terraform workspace new dev
Created and switched to workspace "dev"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
```

만들어 놓은 `dev` 용 input variables 를 사용하여 적용해보자.  

```bash
$ terraform apply -var-file=dev.tfvars
```

적용을 완료하면 도쿄 리전에(ap-northeast-1) 'dev' 이름을 가진 VPC 가 생성된다.

이번에는 `prod` workspace 를 생성하고 적용해보자.  
`dev` workspace 를 생성할때와 동일하게 아래와 같이 생성한다.  

```bash
$ terraform workspace new prod
Created and switched to workspace "prod"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
```

만들어 놓은 `prod` 용 input variables 를 사용하여 적용해보자.  

```bash
$ terraform apply -var-file=prod.tfvars
```

적용을 완료하면 서울 리전에(ap-northeast-1) 'prod' 이름을 가진 VPC 가 생성된다.  

이렇게 main 코드의 분리 없이 환경만 분리하여 작업을 할 수 있으며, `terraform workspace select` 명령어로 적용하고자 하는 workspace만 선택 후 변경사항을 적용하면 된다.  

## 상태관리

그렇다면 `workspace`는 어떤 방식으로 동작하는걸까?  
사실 정말 간단하다.  
위에서 `dev`, `prod` 로 분리하여 서버환경을 구축해봤는데, 해당 폴더의 구조를 한번 살펴보자.  

```bash
├── dev.tfvars
├── main.tf
├── prod.tfvars
├── terraform.tfstate.d
│   ├── dev
│   │   └── terraform.tfstate
│   └── prod
│       └── terraform.tfstate
└── variables.tf
```

`terraform.tfstate.d` 라는 폴더가 생성된게 보이고, 그 아래에 `dev`, `prod` 폴더가 각각 생성되어 상태파일인 `.tfstate` 파일이 관리되고 있는 것을 볼 수 있다.  
이처럼 `workspace` 는 생성된 workspace 의 이름으로 폴더를 만들고 각각 `.tfstate` 파일을 만들어 상태를 관리하게 된다.  
