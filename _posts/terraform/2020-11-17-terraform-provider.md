---
title: "Terraform provider"
excerpt: "Terraform을 사용하기 위해 provider를 설정하자"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Provider

Provider 란 infrastructure 의 type 이라고 생각하면 될 것 같다.  
쉬운 예로 AWS, GCP, Azure 가 각각 provider 라고 생각하면 된다.  
물론 다른 provider 들도 굉장히 많다.  
<https://www.terraform.io/docs/providers/index.html>{:target="_blank"} 여기에서 다양한 provider 들을 확인해 볼 수 있다.  

이런 provider 들에 대해 더 자세히 알아보고 싶다면,
<https://registry.terraform.io/browse/providers>{:target="_blank"} 여기에서 확인할 수 있다.  
이 Site는 Terraform registry 라는 곳인데, 여기에 다양한 Provider 와 Module (Library 라고 생각하면 된다. 추후 해당 내용에 대해 포스팅 하겠다.) 이 등록되어 있고, 해당 내용에 대해 확인할 수 있다.  

자! 그럼 이제 드디어! Provider 를 사용하는 code 를 작성해 보자.  
다양한 Provider 들이 있지만, 앞으로의 실습은 AWS 를 사용하겠다.  
Terraform 의 문법은 동일하니, 다른 Provider 를 사용하고 싶은 사람들도 걱정할 필요는 없다.  

## Provider 설정

```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "3.14.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region     = "ap-northeast-2"
  # Hard-coding credentials is not recommended
  access_key = "XXXXXXXXXXXXXX"
  secret_key = "XXXXXXXXXXXXXX"
}

```

우선 위의 내용을 `example.tf` 라는 파일을 만들어서 복사하자. (`.tf` 파일은 terraform 파일을 뜻한다.)

Terraform 13.0 이상의 버전부터 AWS Provider 를 사용하기 위해서, 위의 code 와 같이 `terraform{}` block에 source 와 AWS provider 의 버전을 선택하면 된다.  
버전을 선택하는 방법은 아래와 같다.  
> `=`: 특정 버전을 명시할 때 사용하며, 다른 condition 과 함께 사용할 수 없다. operator 를 쓰지 않아도 동일하므로 보통은 쓰지 않는다.
>
> ```terraform
> version = "= 3.14.0"
> version = "3.14.0"
> ```
> <br>
> `!=`: 특정 버전을 제외한다. 테스트 해본 결과 특정 버전을 제외 후 이미 install 된 provider 가 있다면 해당 provider 버전을 사용하고, 없다면 특정 버전을 제외한 최신 provider를 install 하여 사용한다.
>
> ```terraform
> version = "!= 3.14.0"
> version = "!= 3.14.0, != 3.14.1, != 3.15.0"
> ```
> <br>
> `>`, `>=`, `<`, `<=`: 특정 버전을 기준으로 높거나 낮은 버전을 선택할 때 사용한다.
>
> ```terraform
> version = "> 3.14.0"  # 3.14.0 초과 버전
> version = ">= 3.14.0" # 3.14.0 이상 버전
> version = "< 3.14.0"  # 3.14.0 미만 버전
> version = "<= 3.14.0" # 3.14.0 이하 버전
> version = "> 3.14.0, <= 3.15.0" # 3.14.0 초과, 3.15.0 이하 버전
> ```
> <br>
> `~>`: 특정 버전과 해당 버전의 가장 낮은 버전 대역의 최신 버전까지 지원하고자 할 때 사용한다. (아래 예제를 살펴보면 이해가 될 것이다.)
>
> ```terraform
> version = "~> 3.14.1" # 3.14.1 이상, 3.15.0 미만 버전 (>= 3.14.1, < 3.15)
> version = "~> 3.14" # 3.14 이상, 4.0 미만 버전 (>= 3.14, < 4.0)
> ```

`terraform{}` block 을 설정하는 법을 살펴봤으니, `provider` block 을 살펴보자.  
`terraform{}` block 에서는 provider 의 `source`와 `version`에 대해 설정을 해줬다면, `provider` block 은 Terraform 이 code 를 수행하기 위해 필요한 정보들을 제공해주는 역할을 한다.  
(`provider` 에서 사용할 수 있는 arguments 에 대해서는 <https://registry.terraform.io/providers/hashicorp/aws/latest/docs#argument-reference> 여기를 참고바란다.)

```terraform
# Configure the AWS Provider
provider "aws" {
  region     = "ap-northeast-2"
  # Hard-coding credentials is not recommended
  access_key = "XXXXXXXXXXXXXX"
  secret_key = "XXXXXXXXXXXXXX"
}
```

`provider` 를 선언하는 곳에 `"aws"` 라고 되어있는데, 이건 위의 `terraform{}` block 에서 `required_providers` 내의 `aws` 라고 선언한 걸 사용하겠다는 의미다.  
Resource 들을 배포하고자 하는 `region` 도 설정해 주고, Terraform 이 명령을 수행하기 위해 반드시 필요한 credentials 정보들도 적어주었다. (본인의 AWS 계정 access key 를 붙혀넣어 사용바란다.)  
Credentials 정보를 제대로 입력해 주었다면, provider 의 설정은 끝이 났다.

:exclamation::exclamation::exclamation: **그런데 잠깐!!**

Local 환경에서 나 혼자 terraform code 를 사용한다면 문제가 없겠지만, 만약 terraform code 를 다른 사람들과 함께 작업하기 위해 Github 에 올린다면..  
위의 코드처럼 작성한다면 AWS credentials 정보까지 모두 source code 에 포함이 되어 공유가 되게 된다. ~~큰일난다.~~

그렇다면 credentials 정보를 어떻게 terraform 에게 전달해 줄 수 있을까?  
다양한 방법이 있는데, 간단한 2가지를 살펴보겠다.

### Credentials 설정 (환경변수)

```bash
$ export AWS_ACCESS_KEY_ID="XXXXXXXX"
$ export AWS_SECRET_ACCESS_KEY="XXXXXXXX"
$ export AWS_DEFAULT_REGION="us-west-2"
```

Terminal 에서 위와 같이 credentials 정보를 환경변수로 입력해준 후,

```terraform
provider "aws" {}
```

이렇게 `provider` 를 생성해준뒤 사용이 가능하다.  
Credentials 를 source code 에서 제거하고 환경변수로 선언하여 사용하는 방법이다.

### Credentials 설정 (AWS credential 파일)

<https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html>{:target="_blank"} 여기를 참조해서 AWS credential 파일을 만든 후, 해당 파일을 terraform 에 제공하여 credentials 정보를 전달해 줄 수 있다.

```terraform
provider "aws" {
  region                  = "us-west-2"
  shared_credentials_file = "/Users/tf_user/.aws/creds" # Credential 파일 위치
  profile                 = "customprofile" # Credential 파일 내의 저장된 profile 중 선택
}
```

## Provider 설치

`.tf` 파일이 있는 곳에서 `terraform init` 명령을 실행해서 provider 를 install 하도록 하자.  

```bash
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "3.14.0"...
- Installing hashicorp/aws v3.14.0...
- Installed hashicorp/aws v3.14.0 (signed by HashiCorp)

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

**Provider** 가 성공적으로 install 된 것을 확인할 수 있다.  
Provider 는 해당 폴더의 `.terraform` 폴더가 생성되어 여기에 install 이 된다.  
(문제가 생겼거나 provider 를 다시 install 하고자 할때는, 간단하게 `.terraform` 폴더를 삭제 후 다시 `terraform init` 명령어를 수행하면 된다.)  
<br>
<br>
자! 지금까지 AWS Provider 를 설정하고 install 하는 방법에 대해 확인해봤다.  
다음 post 에서는 실제로 infrastructure 를 구성하는 `resource` 에 대해 확인해 보겠다.  

<br>
<br>
**[참고]**  

<https://www.terraform.io/docs/configuration/provider-requirements.html>{:target="_blank"}  
<https://www.terraform.io/docs/configuration/providers.html>{:target="_blank"}  
