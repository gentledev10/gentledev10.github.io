---
title: "Terraform modules 사용 방법"
excerpt: "Module 이란 무엇이고 어떻게 사용하는가?"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Modules

## Module 이란?

**Module** 은 한개 또는 여러개의 `.tf` 또는 `.tf.json` 파일들로 구성되어 있으며, module 에 따라 한개 또는 여러개의 resource 들을 포함한다.  
module 내의 파일들은 한개의 directory 에 모두 위치한다.  
module 은 쉽게 생각하면 일종의 library 라고 할 수 있을 것 같다.  
여러 기능들을 포함하고 있는 library 를 만들어서 사용하는 곳에서 원하는 기능들을 사용하듯이,  
여러 resource 들을 포함하고 있는 module 을 만들어서 원하는 곳에서 원하는 resource 들을 생성하거나, 필요한 정보를 가져올 수도 있다.  

module 은 크게 두가지로 나눌 수 있다.  

- **Root module** - Terraform command 를 수행하는 directory 에 있는 파일들로 구성된 module 을 Root module 이라고 한다.  

- **Child module** - 다른 module (Root module 포함) 에서 호출하여 사용되는 module 을 Child module 이라고 한다.  
Child module 은 여러번 호출되어 사용될 수 있고 module 에 따라 다른 configuration 값을 전달하여 사용할 수도 있다.  

## Module 사용 방법

module 을 사용하는 방법에 대해 예제를 통해 먼저 살펴보자.  

<script src="https://gist.github.com/gentledev10/7fd71360ed637b63b909be8a3d714964.js"></script>

module 은 `module` block 을 통해 사용이 가능하며 다른 것과 마찬가지로 unique 한 이름을 함께 명시해주면 된다.  
예제에서는 `example_sg` 란 이름을 사용했으며 나중에 module 의 output 을 참조할때 사용된다.  
block 안의 Arguments 에 대해 하나씩 살펴보자.  

### Module arguments

#### Source

`source` 는 사용하고자 하는 module 을 어디에서 가져올 건지 명시해주는 역할을 한다.  
따라서 모든 module 은 사용시에 반드시 `source` 값을 명시 해줘야 한다.  
같은 source 값을 가지는 `module` block 을 다른 variable 값을 통해 여러번 사용하는 것도 가능하다.  
특정 module 을 처음 사용시에는 `terraform init` 을 통해 해당 module 을 install 하는 과정이 반드시 필요하다.  
`source` 의 값으로 사용할 수 있는 종류에 대해 더 자세히 알아보자.  
- **Local Paths** - local 에 저장되거나 직접 만든 module 을 사용할 수 있다. `./` 또는 `../` 을 사용할 수 있고 해당 module 의 경로를 지정해주면 된다.  
```terraform
module "example" {
    source = "./example"
}
```

- **Terraform Registry** - Terraform 은 Docker hub 같은 module 의 저장소를 제공하는데 바로 [Terraform Registry](https://registry.terraform.io){:target="_blank"} 가 그 역할을 한다.  
Terraform Registry 에는 다양한 module 들이 있고, 모든 module 들은 public 으로 관리가 된다.  
물론 Terraform cloud, Terraform Enterprise 를 사용한다면 Terraform registry 를 private 으로 사용도 가능하다.  
∙∙∙  
AWS security group module 을 사용하는 방법을 예제로 한번 살펴보자.  
먼저 Terraform Registry page 의 검색창에서 *aws security* 라고 검색을 해보자.  
![Terraform registry search module]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-modules/terraform-registry-search-module.png)  
위의 이미지와 같이 Modules 검색 결과에 `terraform-aws-modules/security-group` 이 보일 것이다.  
해당 module 을 클릭하면 아래와 같은 page 를 볼 수 있는데, 아래와 같은 정보들을 얻을 수 있다.  
![Terraform module aws security group]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-modules/terraform-module-aws-security.png)  
1) 인증을 받은 module 임을 나타낸다. 안심하고 사용이 가능하며 되도록 인증 받은 module 을 사용하는 것이 좋다.  
2) Terraform code 내에서 해당 module 을 사용하는 방법에 대해 나와 있다.  
3) Module 의 source code 의 위치가 나와있으면 클릭해서 확인해 볼 수 있다.  
4) Module 의 정보들에 대해 확인할 수 있다. 여기에 나와 있는 정보들을 사용하여 custom 하게 module 을 사용하는 것이 가능하다.  
∙∙∙  
실제 사용을 위해서는 2번에 해당되는 사용방법의 code 를 복사 후 4번에 해당되는 정보들 중 *Inputs* 를 눌러 반드시 필요한 Inputs 2개(`name`, `vpc_id`)를 아래처럼 추가해주면 된다.  
```terraform
module "security-group" {
    source  = "terraform-aws-modules/security-group/aws"
    version = "3.17.0"
    # insert the 2 required variables here
    name   = "example-sg"
    vpc_id = "vpc-xxxxxxxx"
}
```

- **GitHub** - GitHub 에 올려져 있는 module 을 가져와서 사용하는 것도 가능하다.  
아래와 같이 `source` 를 사용하면 HTTPS 를 통해 clone 하여 사용할 수 있고,  
```terraform
module "consul" {
    source = "github.com/hashicorp/example"
}
```  
또는 아래와 같이 사용한다면 SSH 를 통해 clone 하여 사용할 수 있다.  
```terraform
module "consul" {
    source = "git@github.com:hashicorp/example.git"
}
```  

- **Bitbucket** - GitHub 와 마찬가지로 Bitbucket의 사용도 가능하다.  
아래와 같이 `source` 를 정의하여 사용이 가능하다.  
```terraform
module "consul" {
    source = "bitbucket.org/hashicorp/terraform-consul-aws"
}
```  

- **Git Repository** - GitHub 나 Bitbucket 에서 관리되지 않는 git repository 에 대해서도 `source` 로 사용이 가능하다.  
아래와 같은 방법으로 *HTTPS*, *SSH* 를 이용해 git repository의 module을 사용할 수 있다.  
```terraform
// Use HTTPS
module "https" {
    source = "git::https://example.com/example.git"
}
// Use SSH
module "ssh" {
    source = "git::ssh://username@example.com/example.git"
}
```  
∙∙∙  
Git repository 를 module 로 사용시에는 default branch 를 기본으로 사용하게 되는데, `ref` argument 를 사용하여 원하는 *branch* 또는 *tag* 를 사용하는 것도 가능하다.  
아래의 예제를 통해 사용법을 확인해 보자.  
```terraform
// Use other branch
module "other_branch" {
    source = "git::https://example.com/example.git?ref=other_branch"
}
// Use a tag
module "tag" {
    source = "git::https://example.com/example.git?ref=tag_name"
}
```  
`git checkout` 명령을 통해 원하는 branch 또는 tag 로 이동하듯이 `ref` argument 에 원하는 branch 또는 tag 를 입력해주면 된다.  

- **S3 Bucket** - AWS S3 bucket 에 저장되어 있는 module 도 `source` 로 사용이 가능하다.  
```terraform
module "s3_module" {
    source = "s3::https://s3-eu-west-1.amazonaws.com/example-terraform-modules/example.zip"
}
```
위와 같이 `s3` prefix 를 통해 접근이 가능하며, 해당 module 의 install 을 위해서는 AWS credentials setting 이 되어 있어야 한다.  

- **GCS Bucket** - Google Cllud Storage 에 저장된 module 도 `source ` 로 사용이 가능하다.  
```terraform
module "gcs_module" {
    source = "gcs::https://www.googleapis.com/storage/v1/modules/example.zip"
}
```
위와 같이 `gcs` prefix 를 통해 접근이 가능하며, 다음과 같은 규칙을 가진다.  
`gcs::https://www.googleapis.com/storage/v1/BUCKET_NAME/PATH_TO_MODULE`  
GCS 에 저장된 module 의 install 을 위해서는 역시 credentials 가 setting 이 되어 있어야 한다.  

`source` 사용 시 한가지 더 알면 좋은 점은 sub directory 의 module 도 지원한다는 것이다.  
`//` 를 통해 사용이 가능하며 예제를 통해 쉽게 사용법을 알아보자.  
Terraform registry 에 있는 [AWS security group](https://registry.terraform.io/modules/terraform-aws-modules/security-group/aws/latest){:target="_blank"} 를 살펴보면 **Submodules** 라는 메뉴가 보이는데 click 을 해보면 아래와 같이 사용가능한 submodule 들이 나타나다.  

![Terraform module aws security group submodule]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-modules/terraform-registry-submodule.png)  

위의 그림처럼 `http-80` module 을 선택해보자.  

![Terraform module aws security group submodule]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-modules/terraform-registry-submodule-http.png)  

사용법을 확인할 수 있으며 `source` 로 `terraform-aws-modules/security-group/aws//modules/http-80` 를 사용하고 있다.  
`//` 를 통해 security-group module 의 sub directory(modules/http-80) 에 있는 module 에 접근하는걸 확인할 수 있다.  
*source code* 부분을 click 하여 실제 구조가 어떻게 되어 있는지 확인해 보자.  

![Terraform module aws security group submodule]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-modules/terraform-registry-submodule-http-github.png)  

GitHub 에 저장되어 있는 module 을 살펴보면 위의 그림과 같이 `root_directory/modules/http-80` 에 module 이 위치하고 있는 것을 확인할 수 있다.  
물론 GitHub 뿐만 아니라 다른 방식의 source 사용시에도 모두 적용이 가능하며 예제는 아래와 같다.  

- `hashicorp/consul/aws//modules/consul-cluster`  
- `git::https://example.com/network.git//modules/vpc`  
- `https://example.com/network-module.zip//modules/vpc`  
- `s3::https://s3-eu-west-1.amazonaws.com/examplecorp-terraform-modules/network.zip//modules/vpc`  

#### Version

Terraform registry 를 통해 module 을 사용할 경우 `version` 을 명시해 줄 수 있으며, 명시하지 않으면 최신 버전의 module 을 사용한다.  
`version` 에서 사용할 수 있는 값은 이전 [포스트]({{ site.url }}{{ site.baseurl }}/terraform/2020/11/17/terraform-provider.html#provider-설정){:target="_blank"} 에서 확인 가능하다.  
*Terraform registry 를 사용할때만 `verion`을 사용할 수 있음을 반드시 기억하자.*  

#### Meta-arguments

Module 사용시에도 resource 사용과 동일하게 Meta-arguments 를 가지며 `providers` 를 제외하면 동일한 기능을 하기때문에 이전 설명 링크로 설명을 대신하겠다.  

- `depends_on` - [설명]({{ site.url }}{{ site.baseurl }}/terraform/2020/11/29/terraform-resource-2.html#1-depends_on){:target="_blank"}  

- `count` - [설명]({{ site.url }}{{ site.baseurl }}/terraform/2020/11/29/terraform-resource-2.html#2-count){:target="_blank"}  

- `for_each` - [설명]({{ site.url }}{{ site.baseurl }}/terraform/2020/11/29/terraform-resource-2.html#3-for_each){:target="_blank"}  

- `providers` - module 사용 시 어떤 provider 를 사용할건지 명시해 주는 meta-argument 이다.  
만약 명시하지 않는다면 default provider 를 사용해서 module 을 사용하게 된다.  
`resource` 에서 사용시에는 `provider`([설명]({{ site.url }}{{ site.baseurl }}/terraform/2020/11/29/terraform-resource-2.html#4-provider){:target="_blank"})가 한개의 provider 를 전달할때 사용하는 meta-argument 였다면 module 에서는 이름(`providers`)에서 알 수 있듯이 여러개의 `providers` 를 map 형태로 전달할 수 있다.  
map 의 key 는 child module 에서 사용하는 provider 이름이고, value 는 parent module 에서 child module 에 전달할 provider 이다.  
∙∙∙  
몇개의 예제를 통해 사용 방법에 대해 알아보자.  
```terraform
// default provider
provider "aws" {
    region = "us-west-1"
}
// 서울 리전 alternate provider
provider "aws" {
    alias  = "seoul"
    region = "ap-northeast-2"
}
// 도쿄 리전 alternate provider
provider "aws" {
    alias  = "tokyo"
    region = "ap-northeast-1"
}
// default provider 사용 예제
module "default_provider" {
    source = "./default"
}
// 서울 리전 alternate provider 사용 예제
module "alternate_provider" {
    source    = "./alternate"
    providers = {
        aws = aws.seoul
    }
}
// 서울, 도쿄 리전 provider 동시 사용 예제
module "multiple_providers" {
    source    = "./multiple"
    providers = {
        aws.first  = aws.seoul
        aws.second = aws.tokyo
    }
}
```  
**\<Default provider 사용 방법\>**  
"default_provider" module 을 살펴보면 `providers` 가 명시되어 있지 않은데, 이럴 경우 default provider, 즉 위의 예제에서는 "us-west-1" 리전을 child module 에서 사용하게 된다.  
∙∙∙  
**\<Alternate provider 사용 방법\>**  
"alternate_provider" module 을 살펴보면 `providers` 에 child module 의 aws provider 에 parent module 의 aws.seoul provider 를 사용한다고 명시해 주었다.  
이렇게 직접 명시를 하게되면 default provider 대신 명시한 provider 를 사용하게 된다.  
∙∙∙  
**\<여러개의 provider 사용 방법\>**  
자주 있는 상황은 아니지만 만약 child module 에서 여러개의 provider 를 받아 각각 resource 를 생성할 경우 parent module 에서 `provider` 에 여러개의 provider 를 전달할 수 있다.  
"multiple_providers" module 을 살펴보면 `providers` 에 child module 의 aws.first provider 에는 parent module 의 aws.seoul provider 를 사용하고, child module 의 aws.second provider 에는 parent module 의 aws.tokyo provider 을 사용한다고 명시 되어 있다.  
만약 child module 의 code 가 아래와 같다고 생각해보자.  
```terraform
// "first" provider
provider "aws" {
    alias = "first"
}
// "second" provider
provider "aws" {
    alias = "second"
}
// "first" provider 를 사용하여 security group 생성
resource "aws_security_group" "first" {
    provider = aws.first
    name     = "first"
}
// "second" provider 를 사용하여 security group 생성
resource "aws_security_group" "second" {
    provider = aws.second
    name     = "second"
}
```  
각각 first provider 에는 aws.seoul 을, second provider 에는 aws.tokyo provider 를 전달해 주었으므로, `terraform apply` 를 해보면 "first" security group 은 서울 리전에, "second" security group 은 도쿄 리전에 생성 될 것이다.  

#### Input variable arguments

module 사용시에 child module 이 가지고 있는 input variables 가 있다면 parent module 에서 해당 값들을 argument 로 명시하여 configuration 이 가능하다.  
직접 코드를 확인하면 더 이해가 쉬우니 예제를 통해 설명하겠다.  

다음과 같은 child module 이 있다.  
`variables.tf`에 선언된 값들을 이용해 `main.tf` 에서 AWS security group 을 만든다.  

<script src="https://gist.github.com/gentledev10/d8adbfe11b1d4cc42fd6d1514ffe9817.js"></script>  

parent module 에서는 위의 child module 을 아래와 같이 사용할 수 있다.  

<script src="https://gist.github.com/gentledev10/c0f2570a131f1507a920e5c98b637492.js"></script>  

위의 코드에서 확인할 수 있듯이 child module 의 `name`, `description` input variables 가 parent module 에서 argument 로 사용되어 값을 전달받고 있음을 확인할 수 있다.  
`name`의 경우는 필수 값으로 parent module 에서 명시하지 않으면 error 가 발생하지만, `description` 값은 경우에는 optional 값으로 module 사용시에 명시하지 않아도 문제 없이 default 값으로 module 을 사용할 수 있다.  

### Output variables 사용법

Child module 은 사용하는 곳에서 필요한 값들을 제공하기 위해 `output` values 를 정의할 수 있다.  
사용하는 곳에서는 `module.<name>.<output variable>` 방식을 통해 해당 값을 가져올 수 있다.  
이것 또한 코드를 통한 예제를 통해 확인해보자.  

다음과 같은 child module 이 있다.  

<script src="https://gist.github.com/gentledev10/8258f8d34c067f08e2fd32b6d69853f4.js"></script>  

`main.tf` 에서 AWS security group 을 생성하고 `outputs.tf` 에서 `name` output 을 제공해준다.  

parent module 에서는 아래와 같이 "security_group" 이라는 이름으로 module 을 사용하고, child module 에서 "name" 이라는 이름으로 생성한 output 을 `module.security_group.name` 로 접근하여 새로운 "new-example" 이라는 이름의 security group 을 생성하는데 사용하였다.  

<script src="https://gist.github.com/gentledev10/02595ea48d99441c4b721cb853b067a1.js"></script>  

<br>
<br>
지금까지 **Module 의 사용법** 에 대해 알아보았다.  
동일한 module 을 사용자가 원하는 값으로 configuration 할 수 있고, module 을 여러번 재사용 하는 것도 가능하다.  
또한 Terraform registry 를 통해 이미 인증받은 좋은 module 들을 많이 제공하므로 손쉽게 검색 후 사용이 가능하다.  
직접 여러개의 resource 들을 선언하여 생성하기 귀찮거나 관리가 어렵다면, 적절한 module 을 사용하는 것을 추천한다!  
이번 포스트에서는 module 의 사용 방법에 초첨을 맞췄다면, 다음 포스트에서는 어떻게 module 을 직접 만들 수 있는지 알아보도록 하겠다.  

<br>
<br>
**[참고]**  

<https://www.terraform.io/docs/language/modules/syntax.html>{:target="_blank"}
<https://www.terraform.io/docs/language/modules/sources.html>{:target="_blank"}
<https://www.terraform.io/docs/language/modules/develop/providers.html>{:target="_blank"}
