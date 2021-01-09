---
title: "Terraform local values"
excerpt: "Local values 를 사용해보자"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Local values

**Local values** 란 모듈내에서 사용할 수 있는 값이다.  
간단하므로 예제와 함께 살펴보자.  

## 선언 방법

<script src="https://gist.github.com/gentledev10/2f180714b932853ebe5604bd1ea74de1.js"></script>

`locals` block 을 통해 선언할 수 있으며, block 안에 다양한 type 의 variables 를 선언할 수 있다.  
예제에서는 `date`, `common_tags` 두개를 선언하였고 각각 `string`, `object` type 으로 선언해주었다.  
선언 시 다른 local values, resource attributes, varialbes 를 사용하여 선언하는 것도 가능하다.  

## 사용 방법

선언 방법처럼 사용 방법도 굉장히 간단하다.  
`local.<NAME>` 으로 선언한 local value 를 사용할 수 있다.  

위의 예제를 살펴보면 `aws_iam_user.example1`, `aws_iam_user.example2` 의 resource 에서 동일한 tag 값을 가지기 위해 `local.common_tags` 를 사용한 걸 볼 수 있다.  
결국 위의 예제 코드를 `terraform apply` 해보면,  

```bash
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_iam_user.example1 will be created
  + resource "aws_iam_user" "example1" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "test1"
      + path          = "/"
      + tags          = {
          + "created" = "2021.01.09"
          + "group"   = "developer"
        }
      + unique_id     = (known after apply)
    }

  # aws_iam_user.example2 will be created
  + resource "aws_iam_user" "example2" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "test2"
      + path          = "/"
      + tags          = {
          + "created" = "2021.01.09"
          + "group"   = "developer"
        }
      + unique_id     = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_iam_user.example2: Creating...
aws_iam_user.example1: Creating...
aws_iam_user.example2: Creation complete after 2s [id=test2]
aws_iam_user.example1: Creation complete after 2s [id=test1]
```

![Local values result]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-local-values/local-values-result.png)

위와 같이 2개의 IAM User 가 동일한 tag 값이 설정 되는걸 확인할 수 있다.  

<br>
<br>
지금까지 **Local values** 에 대해 살펴보았다.  
선언 방법, 사용 방법 모두 간단하나 잘 사용한다면 중복되는 코드를 제거하거나 코드를 효율적으로 관리하는데 많은 도움이 되니 유용하게 사용하길 바란다.  

<br>
<br>
**[참고]**  

<https://www.terraform.io/docs/configuration/locals.html>{:target="_blank"}
