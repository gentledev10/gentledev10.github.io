---
title: "Terraform dynamic blocks"
excerpt: "Terraform dynamic blocks 의 사용법을 알아보자"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Terraform dynamic blocks

Terraform 을 이용해서 resource 들을 생성하다보면 종종 사용하게 되는게 resource 내부의 configuration block 이다.  
이런 block 들은 한개만 사용될 수도 있지만 경우에 따라서는 여러개를 설정해서 사용하는 경우도 있을 것이고, 한개도 필요 없는 경우도 있을 것이다.  
이런 다양한 경우를 좀 더 유연하게 컨트롤 할 수 있게 해주기 위해 dynamic block 이 사용된다. (0.12 버전부터 지원)  
예제와 함께 사용법에 대해 알아보자.  

## 사용법

예제 코드를 살펴보며 사용법에 대해 익혀보자.  
<https://github.com/gentledev10/terraform_examples/tree/main/dynamic-block>{:target="_blank"}

<script src="https://gist.github.com/gentledev10/aca7713aec029784835933ec43f62dc6.js"></script>

`terraform.tfvars` 파일에서 `ingresses` 와 `egresses` variables 를 정의해서 전달해 주고 있으며, `main.tf` 에서는 VPC 와 security group 을 하나씩 생성해 주고 있다.  
여기서 우리가 살펴볼 부분은 `aws_security_group` resource 내부의 `ingress`, `egress` block 을 생성하는 부분이다.  
코드에서 보는바와 같이 `dynamic` block 으로 정의되어 있으며, 세부 내용에 대해 하나씩 살펴보자.  

- **for_each** - block 을 생성할 정보를 담은 collection 을 전달받고 collection 의 item 수만큼 block 이 생성된다. 예제를 보면 `ingress` 는 https 용 1개의 block을, `egress` 는 http, https 용 2개의 block 을 생성하게 될 것이다.  
또한 아래와 같이 `for` expression 을 사용하여 전달하는 것도 가능하다.  
```hcl
dynamic "egress" {
    for_each = {for k, v in var.egresses: k => {
      fp  = v.from_port
      tp  = v.to_port
    }}
    content {
      description = "${egress.key} description"
      from_port   = egress.value.fp
      to_port     = egress.value.tp
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
}
```
- **iterator** - 기본적으로 for_each 를 통해 전달되는 각각의 item 의 접근을 위해서는 dynamic block 의 label 명으로 접근이 가능하다. 따라서 예제코드에서는 `ingress`, `egress` 로 접근이 가능한데 이걸 변경해주는 역할을 해주는 것이 바로 `iterator`이다.  
`ingress` block 의 예를 들어보면 `iterator`를 ing 로 변경해주었기 때문에 item 접근을 위해 ingress 가 아닌 ing 를 사용하고 있는 걸 확인할 수 있다.  
- **content** - 실제로 정의하고 하는 block 안에 전달되는 값들을 명시하는 곳이다. 각 resource의 terraform 공식 document를 참고하면되고, 예제에서도 `ingress` 와 `egress` block 을 위해서 필요한 값들을 정의해주고 있다.  

## Nested block

resource 중에는 configuration block 안에 또다른 configuration block 을 설정해줘야 하는 경우도 많다.  
이렇게 중첩되는 경우 또한 dynamic block 의 사용이 가능하다.  
예제와 함께 살펴보자.  

<script src="https://gist.github.com/gentledev10/916eccdcf502fc83ca680a713ef073a8.js"></script>

예제 코드를 보면 S3 bucket 생성 시 `lifecycle_rule` 을 dynamic block 으로 설정해주고 있으며, 내부의 `transition` 또한 dynamic block 으로 처리해주고 있음을 확인할 수 있다.  
내부의 dynamic block 은 부모의 content 에 해당하므로 content 내부에 dynamic block 을 선언해주면 되고, `for_each` 에는 예제와 같이 부모의 iterator 를 사용해서 값을 지정해 줄 수도 있다.  
해당 예제는 `terraform.tfvars` 파일에 선언된 것처럼 `example1` lifecycle rule 은 2개의 transition configuration 을 생성할 것이고, `example2` lifecycle rule 은 transition configuration 을 생성하지 않을 것이다.  

## 마무리

Dynamic block 은 configuration block 을 유연하게 사용할 수 있도록 도와준다.  
특히 module 을 만드는 개발자라면 사용자에게 configuration block 을 자유롭게 지정해 줄 수 있도록 해주는 강력한 기능이므로 아마 반드시 필요한 부분이 아닌가 생각된다.  
혹시 모든 configuration block 을 일일이 직접 적어두었다면 dynamic block 을 사용해 보길 추천한다!  

<br>
<br>
**[참고]**  

<https://www.terraform.io/docs/language/expressions/dynamic-blocks.html>{:target="_blank"}
