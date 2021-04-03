---
title: "Terraform cloud 사용법"
excerpt: "Terraform cloud 를 사용해보자"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Terraform cloud

[Terraform cloud](https://www.terraform.io/cloud){:target="_blank"}란 이름 그대로 Terraform 의 운영환경을 cloud 를 통해 관리할 수 있도록 해주는 서비스다.  
예를 들어 Server 를 cloud 환경에서 운영할 수 있도록 해주는 AWS, GCP 같은 서비스라고 생각하면 된다.  
`Terraform cloud` 는 Terraform 의 실행환경과 state 를 관리해주기 때문에, 따로 Terraform 을 위한 instance 를 운영하거나 state 관리를 위해 remote 저장소를 사용하는 일은 하지 않도록 도와준다.  
즉 Terraform 사용자는 Terraform 운영은 모두 `Terraform cloud`에 맡기고, Terraform code 를 통해 infrastructure 를 구축하는 일에 집중할 수 있게된다.  
그렇다면 어떻게 `Terraform cloud` 를 사용하면 되는지 살펴보자.  
`Terraform cloud` 의 다양한 기능에 대해 알아보기 보단 어떻게 구성하고 사용할 수 있는지 실습과 함께 순서대로 알아보겠다.  

## Organization

먼저 `Terraform cloud` 의 계정을 생성하고 로그인을 하면 아래와 같은 화면이 나온다.  

![Welcome to terraform cloud]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/welcome-to-terraform-cloud.png)

`Start from scratch` 를 선택해서 첫 단계부터 시작해보자.  

`Terraform cloud` 의 계정을 생성하고 가장먼저 해야하는 일은 `Organization` 을 생성하는 일이다.  
`Organization` 은 팀과 함께 공유되어 사용되는 공간으로 Terraform 을 수행하기 위한 `Workspace`를 관리한다.  

![Welcome to terraform cloud]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/create-organization.png)

위의 화면처럼 `Organization` 은 이름과 Email 만 입력하면 간단하게 생성할 수 있다.  

***[실습]***  
*Organization 이름과 Email 은 자유롭게 입력하면 된다.*  

## Workspace

`Workspace` 란 Infrastructure 생성을 위한 하나의 set 이다.  
한개의 `Organization`에 여러개의 `Workspace`를 생성하여 사용할 수 있고, 사용방식에 따라 다양한 구성이 가능하다.  
예를 몇가지 들어보면, 아래와 같이 사용이 가능하다.  

- `DEV`, `PROD` 용으로 각각 다른 infrastructure 구성을 위한 2개의 `Workspace`
- `ProjectA`, `ProejctB` 각각 2개의 project 를 위한 2개의 `Workspace`


### 생성
만드는 방법에 대해 살펴보자.  

***[실습]***  
*`Workspace` 생성을 시작하기 전에, 사용할 GitHub Repository 와 code 를 먼저 생성해놓자.  
Repository 내의 `dev`, `prd` 두개의 directory 를 만들고, 각각 아래의 코드파일을 생성한다.*  

*[dev]*
<script src="https://gist.github.com/gentledev10/7cfb18fcbb0ae6eea2bdb892c1bdd336.js"></script>

*[prd]*
<script src="https://gist.github.com/gentledev10/e34ea260e00342a01a1f3b3c3af71b17.js"></script>

*`dev`, `prd` 2개의 `Workspace` 를 만들어 서로 다른 Region 에 VPC 를 생성하기 위한 code 이다.*  
*최종적으로 아래와 같은 구조를 가지면 된다.*  

```bash
├── dev
│   ├── main.tf
│   └── variables.tf
└── prd
    ├── main.tf
    └── variables.tf
```

#### Choose type

`Workspace` 를 생성하기 위한 첫번째 단계는 어떤 type 의 `Workspace` 를 만들지 결정하는 단계이다.  
GitHub, GitLab, Bitbucket, Azure DevOps 와 연동이 가능한 `Version control woorkflow` 가 대부분의 경우 가장 많이 사용된다.  
이 외의 CLI 또는 API 를 이용하는 `Workspace` 의 구성도 가능하다.  

***[실습]***  
*Github 의 Terraform code 와 연동하는 `Workspace` 를 만들기 위해 `Version control workflow` 를 아래와 같이 선택한다.*  

![Workspace choose type]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-choose-type.png)

#### Connect to VCS

사용하고자 하는 VCS(Version control provider) 를 선택하는 단계이다.  
본인이 사용하고자 하는 VCS 를 선택해서 연동하면 되고 연동된 VCS 의 Repository 의 목록을 가져오는 것이 가능해진다.  

***[실습]***  
*Github 를 선택하여 연동해보자.*  

![Workspace connect to VCS]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-connect-vcs.png)

*Github 를 선택하면 아래와 같이 Terraform cloud 에서 Repository 에 접근을 위한 인증을 요구하므로 승인을 해준다.*  

![Workspace authorize]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-authorize.png)

*권한을 승인하면 이제 Terraform cloud 에서 Repository 목록을 가져올 수 있고, Terraform cloud 와 연동할 Repository 를 선택하여 `Install` 버튼을 누르면 연동이 완료된다.*  

![Workspace repository install]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-repository-install.png)

#### Choose a repository

Repository 와 연동을 마쳤다면 이제 `Workspace` 에서 사용할 Repository 를 선택하는 단계이다.  
연동된 Repository 목록이 화면에 나타나고 사용할 Repository 를 선택하면 된다.  

![Workspace choose repository]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-choose-repository.png)  

***[실습]***  
*위에서 미리 준비한 Repository 를 선택한다.*  

#### Configure settings

마지막 단계는 상세 설정을 하는 단계이다.  
각각의 설정에 대해 하나씩 알아보자.  

![Workspace configure settings]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-configure-settings.png)  

- `Workspace Name` - Workspace 의 이름
- `Description` - Workspace 의 설명
- `Terraform Working Directory` - 선택한 Repository 의 최상단이 아닌 다른 directory 를 지정하여 Terraform 을 수행하고 싶을때 사용할 수 있는 setting 이다.  
즉 Terraform 이 수행될 root directory 를 지정해주는 setting 이다.
- `Automatic Run Triggering` - Terraform 수행을 항상 할 것인지, 특정 path 내의 file 이 수정된 경우에만 할 것인지 정하는 setting 이다.  
특정 path 옵션을 선택하면 여러개의 path 를 추가로 입력할 수 있다.  
- `Automatic speculative plans` - Pull request 시에 `Terraform plan` 을 trigger 할건지 선택하는 setting 이다.  
Pull request 의 Review 전에 `Terraform plan` 의 결과를 확인해 볼 수 있어 편리하다.  
- `VCS branch` - Terraform 을 수행할 branch 가 default branch 가 아닌 경우 변경이 가능하다.  
- `Include submodules on clone` - Git submodule 을 사용중이라면 submodule 까지 clone 해줄 수 있는 setting 이다.  

***[실습]***  
*`dev` 용 `Workspace` 를 위한 설정을 먼저 진행해보자.*  

- `Workspace Name` - 자유롭게 지정
- `Description` - 자유롭게 설명 추가
- `Terraform Working Directory` - `dev` 입력 (`/dev` 폴더를 root directory 로 사용하겠다는 뜻이다.)
- `Automatic Run Triggering` - `dev` 폴더 내의 파일이 변경되었을때만 Terraform 수행이 필요하므로 `Only trigger runs when files in specified paths change` 를 선택한다.  
자동으로 `Terraform Working Directory` 으로 지정한 path 가 선택이 되며, 필요에따라 path 의 추가도 가능하다.
- `Automatic speculative plans` - Pull request 시에 `Terraform plan` 결과를 볼 수 있도록 옵션을 킨다.
- `VCS branch` - default branch 를 사용할 것이므로 그냥 비워두면 된다.
- `Include submodules on clone` - Submodule 을 사용하지 않으므로 체크하지 않는다.

*모든 입력이 끝난 후 `Create workspace` 버튼을 통해 `Workspace` 생성을 완료한다.*  
*`prd` 용 `Workspace` 도 `Workspace Name` 과 `Terraform Working Directory` 를 `prd` 로 사용하는 변경만 한뒤 동일한 방식으로 생성하면 된다.*  

### Variables 설정

Terraform 수행 시 필요한 Input variables, 또는 실행환경에서의 필요한 환경변수를 code 내에서 관리하는 것이 아닌 Terraform cloud 내에서 `Workspace` 별로 설정하는 것이 가능하다.  
생성된 `Workspace` 를 선택하고 나타나는 메뉴 중 `Variables` 를 선택하면 된다.  
Variables 를 입력할때는 추가적으로 2가지 옵션이 제공된다.  

- `HCL` - 기본적으로 여기에 입력되는 Variables 는 String 으로 인식이 되지만 이 옵션을 체크하면 HCL 로 인식하여 parsing 을 하게된다.  
즉 interpolation(${example})의 사용도 가능해진다.
- `Sensitive` - 숨기고자 하는 값을 설정할때 사용된다. 이 옵션을 선택하면 value 의 값이 보이지 않게 설정된다.  

***[실습]***  
*실습을 위해 생성한 Terraform code 를 수행하기 위해서는 `region` Input variables 와 AWS 연동을 위한 정보가 필요하다.*  
*아래와 같이 `Terraform Variables` 쪽에 `region` 값을, `Environment Variables` 쪽에 AWS 연동 정보를 `Sensitive` 옵션과 함께(비밀정보이므로) 추가해준다.*  

![Workspace variables]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-variables.png)

*`dev`, `prd` 각각 region 을 다르게 설정해보자.*  

### Terraform 수행

`Workspace` 및 Variables 까지 설정이 완료됐다면 Terraform 을 수행할 차례이다.  
실습과 함께 방법에 대해 알아보자.  

***[실습]***  
*`Workspace` 를 선택하고 `Queue plan manually` 를 선택한다.*  

![Workspace variables]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-run.png)

*이후 나타나는 창에 `Reason` 을 작성한 후 `Queue plan` 을 선택하면 Terraform 수행이 시작된다.*  
*`Workspace`의 `Runs` 메뉴에서 `Current Run` 을 보면 Terraform 이 수행되고 있는걸 확인할 수 있다.*  
*아래와 같이 Terraform plan 을 먼저 수행하여 변경사항에 대해 보여주고, `Confirm & Apply` 버튼을 클릭해 적용까지 완료할 수 있다.*  

![Workspace variables]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-apply.png)

### 변경사항 적용

GitHub 에서 관리되고 있는 Terraform code 가 변경되었을때는 어떤 방식으로 Terraform cloud 에서 적용이 되는지 실습과 함께 알아보자.  

***[실습]***  
*생성한 `Workspace` 와 연동한 GitHub repository의 `dev/main.tf` 파일에 다음과 같이 Subnet 을 생성하는 코드를 추가하여 PR 을 생성한다.*  

<script src="https://gist.github.com/gentledev10/50cdab75d9b07f91daca39af5a33c4ff.js"></script>

*`Workspace` 의 설정에서 `Automatic speculative plans` 가 선택되어 있다면, PR 생성과 동시에 webhook 을 통해 해당 PR 의 `Terraform plan` 이 수행되고 리뷰어들은 코드리뷰와 동시에 `Terraform plan` 의 결과물까지 확인할 수 있다.*  

![Workspace terraform plan]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-terraform-plan.png)

*이제 PR 을 merge 하고 해당 `Workspace`의 `Runs` 탭을 살펴보자.*  
*Merge 됨과 동시에 해당 `Workspace`에서 Infrastructure 반영을 위해 Terraform 을 수행한다.*  

![Workspace runs]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-runs.png)

*`Confirm & Apply` 를 선택해 Apply 까지 완료하면 아래와 같이 초록색으로 `APPLIED`로 표시되어 있는걸 확인할 수 있다.*  

![Workspace applied]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-run-applied.png)

*이와 같은 절차로 Terraform code 의 변경사항을 Terraform cloud 내에서 쉽게 적용할 수 있다.*  

### State 확인

각각 `Workspace` 별로 현재 Infrastructure 의 state 를 가지고 있는데, `States` 탭에서 쉽게 확인이 가능하다.  
`States` 탭을 누르면 Terraform 이 수행될때마다 그 시점의 state 를 보관하며 클릭을 해보면 어떤것이 변경됐는지에 대해서도 확인이 가능하다.  

![Workspace states]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-cloud/workspace-states.png)

### Destroy 및 Delete

생성된 Infrastructure 를 모두 destory 하거나 `Workspace` 를 삭제하는 일도 쉽게 가능하다.  
`Settings` 탭의 `Destruction and Deletion` 을 선택하면 되고 `Queue destroy plan` 을 통해 Infrastructure 의 destroy를, `Delete from Terraform Cloud` 를 통해 `Workspace`의 삭제가 가능하다.  
Destroy 시에는 바로 삭제가 되는건 아니고 Terraform code 변경을 통한 적용처럼 `Runs` 탭에 destroy 를 위해 Terraform 이 수행되고 `Confirm & Apply` 를 통해서 최종 적용이 된다.  

## 마무리

지금까지 기본적인 Terraform cloud 의 사용법에 대해 알아보았다.  
팀원들과 함께 Terraform을 관리한다면 안정적이고 편리하게 사용이 가능한 Terraform cloud 를 강력 추천한다!  

<br>
<br>
**[참고]**  

<https://www.terraform.io/docs/cloud/workspaces/index.html>{:target="_blank"}
<https://www.hashicorp.com/products/terraform/pricing>{:target="_blank"}
