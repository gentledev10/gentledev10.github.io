---
title: "Terraform output variables"
excerpt: "Output variables 를 사용해보자"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Output variables

**Output variables** 란 모듈에서 제공하는 값이며, 다른 모듈에서 그 값을 다양하게 사용하여 Infrastructure 구성에 도움을 준다.  
모듈의 return value 라고 생각하면 쉬울 것 같다.  

## 선언 방법

예제와 함께 선언 방법에 대해 알아보자.

<script src="https://gist.github.com/gentledev10/37f814311d41bf9729b2ac71c40bbdb2.js"></script>

Output variables 는 보통 `outputs.tf` 파일을 생성하여 선언하며 (물론 파일 이름은 개발자 자유이다) `output` block 을 통해 생성할 수 있다.  
output 의 이름은 다른 것들과 마찬가지로 다른 output 의 이름과 중복 되어서는 안된다.  

사용할 수 있는 Arguments 에 대해서 살펴보자.  
`value`를 제외한 모든 Arguments 는 optional 이므로 사용하지 않아도 문제는 없다.  

- `value` - output 으로 사용할 값을 명시해준다.  string, number, list 등등 다양한 type 을 가질 수 있다.  

- `description` - output 에 대한 설명이다.  
만약 모듈을 만들어 배포한다면 `description` 에 적은 내용이 문서에 나타나 모듈 사용자들이 참고하게 되므로 사용자를 위한 내용을 적는 걸 추천한다.  
해당 모듈 개발자를 위한 설명은 주석으로 달아도 충분하다.  
<br>
AWS VPC module 을 살펴보고 어떻게 사용되는지 알아보자.  
[AWS VPC module](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest){:target="_blank"} page 에서 아랫부분에 있는 **Outputs** list 를 살펴보자.  
이 list 에 있는 `Description` 이 해당 모듈의 [outputs.tf 파일](https://github.com/terraform-aws-modules/terraform-aws-vpc/blob/master/outputs.tf){:target="_blank"} 의 `description`의 내용임을 확인할 수 있다.  

- `depends_on` - 일반적으로 Terraform 이 알아서 처리해주기 때문에 필요가 없지만 특별한 경우 dependencies 가 필요할때 `depends_on` 을 사용하여 처리 순서를 보장할 수 있다.  

## 사용 방법

**Outputs** 를 사용하는 방법은 크게 두가지가 있다.  

1. Root module 에서는 `terraform apply` 를 통해 infrastructure 가 구성된 후 정의된 outputs 들이 command line 창에 나타나게 된다.  

2. Child module 에서는 outputs 들이 attribute 로 root module 에 전달된다.  

우선 모듈에 대한 포스팅은 추후 올릴 예정이므로, 현재 설명이 가능한 1번 경우에 대해서만 확인해 보겠다. (2번의 경우는 모듈에 대한 포스팅에서 다시 다뤄보겠다.)  

위에 있는 예제 파일을 생성하고 `terraform init` 이후 `terraform apply` 를 실행해 보자.  
아래와 같은 출력 결과를 얻을 수 있다.  

```bash
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_iam_user.example will be created
  + resource "aws_iam_user" "example" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "test"
      + path          = "/"
      + unique_id     = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + custom_name = "custom-test"
  + hello       = "hello"
  + user_arn    = (known after apply)
  + user_name   = "test"

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_iam_user.example: Creating...
aws_iam_user.example: Creation complete after 2s [id=test]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

custom_name = custom-test
hello = hello
user_arn = arn:aws:iam::111111111111:user/test
user_name = test
```

`terraform apply` 명령어를 통해 "test" IAM user 를 하나 만들었고, 그 이후 코드에서 선언해 놓은 4개의 outputs 가 출력된 걸 볼 수 있다.  
하나씩 살펴보자.  

- `user_name` - `aws_iam_user.example.name` 을 통해 생성된 user 의 name 을 가져와서 출력해 주었으므로 **test** 가 출력되었다.

- `user_arn` - `aws_iam_user.example.arn` 을 통해 생성된 user 의 arn 을 가져와서 출력해 주었으므로 **arn:aws:iam::111111111111:user/test** 가 출력되었다.  

- `custom_name` - `custom-` string 에 `aws_iam_user.example.name` 을 더한 결과를 출력해 주었으므로 **custom-test** 가 출력되었다.  

- `hello` - 단지 hello 라는 string 을 출력해주는 output 이므로 **hello** 가 출력되었다.  

사용법은 이렇게 간단하고 `value` 에는 다양한 type 이 가능하므로 필요에 맞게 설정 후 사용하면 된다. (함수 사용도 가능하므로 함께 이용하면 원하는 output 을 출력하는데 도움이 될 것이다.)  

<br>
<br>
지금까지 **Output values** 에 대해 살펴보았다.  
Terraform 을 통한 infrastructure 구성 후 필요한 정보들을 output 을 통해 관리한다면 귀찮게 찾아보는 일 없이 쉽게 정보 획득이 가능할 것이다.  

<br>
<br>
**[참고]**  

<https://www.terraform.io/docs/configuration/outputs.html>{:target="_blank"}
