---
title: "Terraform data sources"
excerpt: "Data source 에 대해서 알아보자."
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Data sources

**Data source** 는 Terraform 을 사용하지 않고 만든 infrastructure resource 또는 다른 곳에서 사용중인 Terraform code 를 통해 만들어진 resource 의 Data 를 가져오는데 사용된다.  
각각의 provider 들은 resource 와 함께 data source 도 제공하고 있다.  

## Data source 문법

예제를 통해 어떻게 사용하는지 알아보자.  

<script src="https://gist.github.com/gentledev10/0b987d17446a905a2aadad6cd0e00cb8.js"></script>

코드의 첫번째 라인처럼 `data` block 을 통해 원하는 type 의 data 를 가져올 수 있다.  
예제 코드에서는 `user_name` 이 `foo_name` 인 `aws_iam_user` 의 data 를 가져오도록 하였고 `foo` 라는 이름을 붙혀줬다.  

이처럼 data 를 가져오기 위해서는 `data` block 안에 정보들(Arguments)을 명시해줘야 하는데, 이 Arguments 에 대한 내용은 각각의 Provider document 페이지에서 확인할 수 있다.  
예제에서 사용한 `aws_iam_user` data 같은 경우도 [document page](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_user#argument-reference){:target="_blank"} 에서 확인할 수 있으며, 아래와 같다.  

![Argument Reference]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-data-source/data-argument-reference.png)

`resource` 사용법과 동일하게 ***(Required)*** Argument 에 대해선 반드시 명시를 해줘야하고, ***(Optional)*** 인 경우에는 필요한 경우만 명시하여 사용하면 된다.  
또한 `resource` 사용법과 동일하게 *data type* 과 *data name* 을 합친건 unique id 로 사용되므로 다른 것과 중복되면 안된다.  

## Data instance 참조

`data` block 을 통해 data instance 가 생성되고 우리는 이걸 참조하여 원하는 data 를 가져올 수 있다.  
`resource` 사용법과 동일하게 `data.<TYPE>.<NAME>.<ATTRIBUTE>` 문법을 사용하여 가져올 수 있으며, 위의 예제에서는 `resource` block 내에서 `data.aws_iam_user.foo.path` 를 사용하여 foo iam user 의 path 값을 가져왔다.  

각각의 data instance 는 type 에 따라 참조할 수 있는 Attribute 가 있으며, 각각의 data source 문서를 참고하면 된다.  
여기서 사용한 `aws_iam_user` 의 [document page](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_user#attributes-reference){:target="_blank"} 를 참고해보면 아래와 같은 Attribute 가 제공됨을 알 수 있다.

![Argument Reference]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-data-source/data-attribute-reference.png)

## filter block

**filter** 란 용어 그대로 가져온 Data instance 중 원하는 Data instance 를 얻고자 할때 사용된다.  
예제를 통해 살펴보자.  

<script src="https://gist.github.com/gentledev10/301039a48ba592a24be3575b2650d873.js"></script>

`data` block 을 통해 `aws_ami` 를 가져오고 있으며, `"name"` 이란 filter 를 통해 `"amzn2-ami-hvm*"` 에 해당하는 Data instance 를 가져온다. (이와 같이 정규식의 사용도 가능하다.)  
`most_recent` 와 함께 사용하였으므로 이 `data` block 을 통해 가장 최신의 Amazon Linux 2 AMI 를 가져올 수 있다.  
Filter 에서 사용할 수 있는 key 값은 `aws_ami` [Attribute document page](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ami#argument-reference){:target="_blank"} 에서 `filter` 부분을 보면 확인할 수 있다. ([describe-images 를 위한 AWS CLI Reference](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ami#argument-reference){:target="_blank"}의 --filter option 을 살펴보라고 나와있다.)  
예제에서 이렇게 가져온 Data instance 를 통해 `aws_instance` resource 를 생성하는 부분까지 확인해 볼 수 있다.  

`filter` 는 아래 예시와 같이 여러개의 block 을 명시한다면 **AND** type 으로 동작하고,  

```terraform
filter {
  name   = "name"
  values = ["amzn2-ami-hvm*"]
}

filter {
  name   = "virtualization-type"
  values = ["hvm"]
}
```

아래 예시와 같이 한개의 block 에서 여러개의 values 를 명시한다면 **OR** type 으로 동작한다.  

```terraform
filter {
  name   = "name"
  values = ["amzn2-ami-hvm*", "ubuntu-*"]
}
```

## Local-only data sources

일부 Data source 중에는 remote infrastructure 에서 data 를 가져오는 것이 아니라, local 에서 처리한뒤 data instance 를 생성하는 것들이 있다.
`template_file`, `local_file` `aws_iam_policy_document` 이 3가지가 있는데, 간략한 설명과 공식 documentation page 를 적어놓을테니 사용법은 여기서 확인 바란다.  

> **[template_file]** - [document page](https://registry.terraform.io/providers/hashicorp/template/latest/docs/data-sources/file){:target="_blank"}
> `.tpl` 파일인 template file 을 가져와서 data instance 로 만들어 rendered template 을 참조하고자 할때 쓰는 data source 이다.  
> **하지만 Terraform 0.12 이상 버전부터 deprecate 되었고** `templatefile` function 을 사용해서 동일한 동작을 수행할 수 있으므로 0.11 이하의 버전을 사용하지 않는다면 딱히 쓸일은 없다.  
>  
> **[local_file]** - [document page](https://registry.terraform.io/providers/hashicorp/local/latest/docs/data-sources/file){:target="_blank"}
> Local 에 있는 file 을 data instance 로 생성하여 해당 파일의 content 또는 base64 로 encoding 된 content 를 참조할 수 있도록 해주는 Data source 이다.  
>  
> **[aws_iam_policy_document]** - [document page](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document){:target="_blank"}
> `aws_iam_policy` resource 를 생성할때 사용할 policy 를 생성해주는 Data source 이다.  
> Arguments 들을 통해 data instance 를 생성하면 json 으로 변환된 값을 참조할 수 있다.  

<br>
<br>
지금까지 **Data sources**에 대하여 살펴보았다.  
`resource` 와 함께 Terraform 의 필수 요소 중 하나이며, 잘 사용한다면 더욱 안정적인 Infrastructure as Code 를 경험할 것이다.
