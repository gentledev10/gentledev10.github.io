---
title: "Terraform module publishing"
excerpt: "Module 을 직접 publishing 해보자"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Module publishing

내가 만든 module 을 혼자 사용하는 것도 좋지만 전세계 사람들과 함께 공유하여 사용한다면 어떨까?  
잘 만든 module 을 통해 다른 Terraform 개발자들이 Infrastructure 를 구성하도록 도와줄 수 있다면 아주 멋진 일이 될 것이다.  
그렇다면 어떻게 다른 개발자들에게 공유할 수 있을까?  
Terraform 에서는 [Terraform registry](https://registry.terraform.io/){:target="_blank"} 에 누구나 쉽게 자신이 만든 module 을 publish 할 수 있게 해준다.  
Publish 된 module 은 전세계 개발자들이 누구나 쉽게 검색하여 사용이 가능하다.  

## Module publish 요구사항

Module 을 publish 하기 위해서는 몇가지 따라야 할 요구사항이 필요하다.  

* **GitHub** - 개발한 module 을 public Terraform registry 에 배포하기 위해서는 반드시 GitHub public repo 에 있어야 한다. (물론 private registry 에 배포한다면 이 요구사항은 필요 없다.)  

* **Repository 이름** - GitHub 의 repository 이름은 다음과 같은 규칙으로 생성되어야 한다. `terraform-<PROVIDER>-<NAME>`. `<PROVIDER>` 에는 Terraform 에서 사용할 provider 이름(aws, google 등등)이 사용되고, `<NAME>` 에는 module 명으로 사용할 이름을 쓰면 된다. `<NAME>` 에는 `terraform-aws-vpc-example` 의 `vpc-example` 부분 처럼 추가적인 hyphen 을 사용하는 것도 가능하다.  

* **README.md 파일** - README.md 파일에 해당 module 에 대한 설명이 필요하다. 사실 이 파일이 없더라도 module 을 publish 하는데는 문제가 없지만, module 설명 탭(Readme) 에 README.md 가 없어 이 module 은 internal 로 사용되는 걸로 보인다고 나타나므로 반드시 함께 포함시켜주는 것이 좋다.  
(README.md 가 없을 시 아래와 같은 문구가 나타난다!) ![No README.md error]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-module-publish/terraform-registry-without-readme.png)

* **기본 Module 구조** - Terraform 에서 권장하는 기본 module 구조를 따라야 한다. Terraform registry 에서 module publish 할때 input, output 등의 document 를 code 에서 읽어 자동으로 생성해 주는데 권장 하는 기본 module 구조를 따르지 않으면 문제가 발생하게 된다. [Standard module structure](https://www.terraform.io/docs/language/modules/develop/structure.html){:target="_blank"} 페이지에서 module 구조에 대해 확인할 수 있다.

* **Git tag** - Terraform registry 는 module 의 버전 관리를 위해 git tag 를 사용하며 tag 는 반드시 [semantic version](http://semver.org/){:target="_blank"} 형식이어야 한다. 따라서 tag 는 `1.0.0`, `1.0.2-alpha` 와 같은 형식이 될수 있고, prefix 로 `v` 를 사용해 `v1.0.0` 같은 형식도 허용한다.

## Module 을 publish 해보자

Module publish 를 위한 요구사항에 대해 알아보았으니, 위의 요구사항에 따라 module 을 만들고 publish 를 직접 해보도록 하겠다.  

### GitHub repository 생성 및 코드 작성

본인의 GitHub 에 AWS VPC module 을 위해 `terraform-aws-vpc` 이름으로 repository 를 생성하고 코드를 작성한다.  
코드는 <https://github.com/gentledev10/terraform-aws-vpc>{:target="_blank"} 에서 확인 및 복사해서 사용하면 된다.  
Repository 와 code 까지 완성이 되었다면 추가적으로 해야할 일은 tag 를 달아주는 것이다.  
Terraform registry 는 tag 를 가지고 version 을 관리하기 때문에 반드시 tag 가 명시되어야 한다.  
예제에서는 `v1.0.0` tag 를 달아주었다.  

### Terraform registry sign-in

Module 을 Terraform registry 에 배포하기 위해서는 먼저 Terraform registry 에 sign-in 을 하고 authorize 하는 과정이 필요하다.  
[Terraform registry](https://registry.terraform.io/){:target="_blank"} 에서 오른쪽 상단의 `Sign-in` 을 선택하고  

![Sign-in]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-module-publish/terraform-registry-signin-1.png)

그다음 나타나는 화면에서 `Sign in with GitHub` 를 선택해서 GitHub 와 연결시켜 주는 작업을 진행한다.  

![Sign-in]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-module-publish/terraform-registry-signin-2.png)

Sign-in 을 하게되면 Terraform registry 에서 나의 GitHub 에 Access 하여 module 을 가져올 수 있도록 해줘야 하는데 아래 그림처럼 `Authorize hashicorp` 를 선택해주면 된다.  

![Sign-in]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-module-publish/terraform-registry-authorize.png)

지금 까지의 단계로 이제 Terraform registry 에 module 을 배포할 수 있는 준비가 되었다.  

### Module publishing

이제는 내 GitHub 에 생성했던 `terraform-aws-vpc` module 을 publish 해보자.  
Terraform registry 의 오른쪽 상단의 메뉴를 보면 `Publish` 가 있고 sub 메뉴인 `Module` 을 선택하자.  

![Sign-in]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-module-publish/terraform-registry-module-publish.png)

다음화면에서 publish 하고자 하는 module 을 선택할 수 있는데, 위의 요구사항에서 확인했듯이 `terraform-<PROVIDER>-<NAME>` naming 에 부합하는 repository 만 list 에 나와서 선택할 수 있다.  
우리가 만들었던 `terraform-aws-vpc` 를 선택하고 `PUBLISH MODULE` 를 클릭하면 배포가 완료된다!  

![Sign-in]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-module-publish/terraform-registry-module-select.png)  

### 배포된 module 확인

Module 의 publish 가 완료되면 해당 module 에 대한 page 가 생기고, 자동으로 생성된 문서를 확인할 수 있다.  
이제 module 배포가 되었으므로 Terraform registry 에서 우리가 배포한 module 의 검색도 가능하다.  
문서에는 총 5개 탭이 있는데 간단하게 하나씩 살펴보자.  

* **Readme** - 우리가 생성한 README 파일의 내용을 보여준다.  

* **Inputs** - `variables.tf` 을 통해 만들어진 input variables 들을 이용하여 자동으로 생성된 문서가 보여지는 탭이다.  

* **Outputs** - `outputs.tf` 을 통해 만들어진 output 들을 이용하여 자동으로 생성된 문서가 보여지는 탭이다.  

* **Dependency** - 해당 Module 이 가지고 있는 dependency 들을 자동으로 생성된 문서로 보여주는 탭이다. 다른 외부 module 의 사용 여부나 provider 의 version dependency 같은 정보들을 보여준다.  

* **Resources** - Module 을 통해 생성되는 resource 들의 list 를 보여준다.  

### (선택사항) Examples 추가하기

Terraform registry 에서는 배포된 module 의 사용법을 Examples 로 보여줄 수 있는 기능을 제공한다.  
선택사항으로 module 배포시에 반드시 해야하는 건 아니지만 사용자들에게 해당 module 을 직접 사용하는 다양한 방법을 보여줌으로써 도움을 줄 수 있으므로 방법에 대해 한번 알아보자.  

#### 공식 AWS VPC module examples 확인

먼저 [공식 AWS VPC module 페이지](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest){:target="_blank"}에서 어떻게 Examples 를 제공하는지 살펴보자.  
아래와 같이 페이지에서 **Examples** 라는 메뉴를 제공한다.  

![Official AWS VPC module]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-module-publish/terraform-registry-aws-vpc.png)  

클릭하면 아래와 같이 여러개의 examples 가 나오고 원하는 example 을 선택하여 정보를 확인할 수 있다.  

![Official AWS VPC module examples]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-module-publish/terraform-registry-aws-vpc-examples.png)  

`complete-vpc` 를 선택해서 정보를 확인해보자.  
아래와 같이 `complete-vpc` example 에 대한 정보가 나오고 가장 중요한 건 Source code 위치를 제공해 어떻게 module 을 사용했는지를 쉽게 확인할 수 있다.  

![Official AWS VPC module complete-vpc example]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-module-publish/terraform-registry-aws-vpc-examples-complete.png)  

#### Example 작성

자! 그럼 우리가 배포한 module 에도 example 을 추가해보자!  
Examples 제공을 위해서는 배포한 module 의 GitHub repository 에 `examples/<EXAMPLE_NAME>` 으로 폴더를 만들고, 거기에 `.tf` 파일을 만들어 사용방법에 대한 코드를 작성하면 된다.  
`examples/` 폴더 안에 제공하고자 하는 다양한 examples 들에 대한 폴더를 만들어 제공하는 것이 가능하다.  
우리가 배포한 module 의 사용법을 위해 간단하게 `examples/vpc` 폴더를 만들어서 사용하는 코드를 작성해보자.  
실습을 위해 [여기](https://github.com/gentledev10/terraform-aws-vpc/tree/main/examples/vpc){:target="_blank"}에 있는 코드를 사용하면 되고, 코드가 merge 된 후 tag 까지 배포하면 완료된다.  
일정시간 후에 자동으로 우리가 만든 module 페이지가 업데이트 되지만 빠른 확인을 위해 module 페이지에서 `Manage Module` 메뉴의 `Resync Module` 을 누르고 페이지를 새로고침하면 다음과 같이 **Examples** 메뉴가 나타나고 우리가 작성한 `vpc` example 이 생성된 것을 확인할 수 있다.  

## 마무리

지금까지 우리가 만든 module 을 Terraform registry 에 public 하게 배포하는 방법에 대해 알아보았다.  
Docker image 를 public 하게 공유하여 사용하듯, 이 방법을 통해 Terraform module 도 전세계 사람들과 함께 공유하여 사용하는 것이 가능하다.  
본인이 좋은 module 을 만들어 사용중이라면 Terraform registry 에 공유해보는 것을 추천한다!

<br>
<br>
**[참고]**  

<https://www.terraform.io/docs/registry/modules/publish.html>{:target="_blank"}
