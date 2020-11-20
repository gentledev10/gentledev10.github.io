---
title: "Terraform resource (1/2)"
excerpt: "Resource 를 이용해 infrastructure 를 구성해보자"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Resource

**Resource** 는 Terraform 의 가장 중요한 구성요소 중 하나이다.  
Resource 를 선언함으로써 virtual networks, compute instance 등 다양한 infrastructure 를 생성할 수 있다.  
바로 실습을 하며 어떻게 사용하는지 알아보자! 쉽게 배울 수 있다.  

## Resource 문법

간단한 예제를 통해 resource 문법에 대해 알아보자.
<script src="https://gist.github.com/gentledev10/a5dbbcaff63e36de29d23789ba49a3c6.js"></script>

위의 간단한 예제에서 보이듯이 `resource` 라는 block 을 만들어서 원하는 resource 를 생성할 수 있다.  

### Type, Name

`"aws_iam_user"` 라고 선언된 부분은 *resource type* 에 해당되는 부분으로 예제에서는 AWS 의 IAM User 를 생성하겠다는 뜻이다.  
이 resource type 은 어떤 provider 를 사용하냐에 따라 사용할 수 있는 type 들이 달라진다. (각 provider 의 documentation 페이지에 모든 resource type 들이 나와 있다.)  
`"example"` 이라고 선언된 부분은 *resource name* 에 해당되는 부분으로 특정 resource type 의 이름을 선언해준다.  

*resource type* 과 *resource name* 을 합친건 module(혹시 모르는 분은 library 라고 생각하면 된다.) 내에서 unique 하게 사용되므로, 중복되는 값을 가질 수 없다.  
예제를 통해 알아보자.

> **[사용가능]**
>
> ```terraform
> resource "aws_iam_user" "example" {
>   name = "example"
> }
>
> resource "aws_iam_user" "example2" {
>   name = "hi"
> }
>
> resource "aws_vpc" "example" {
>   cidr_block = "10.0.0.0/16"
> }
> ```
>
> `"aws_iam_uer"` resource type 이 2번 사용되었는데, 2개가 각각 다른 이름을("example", "example2") 가지고 있으므로 문제가 없다.  
> `"aws_vpc"` resource type 은 기존에 사용된 `"example"` 이라는 이름과 함께 사용되었지만 서로 다른 type 에서 사용되었기 때문에 문제가 없다.
>
> **[사용 불가능]**
>
> ```terraform
> resource "aws_iam_user" "example" {
>   name = "example"
> }
>
> resource "aws_iam_user" "example" {
>   name = "hi"
> }
> ```
>
> 동일한 `"aws_iam_user"` 에서 동일한 이름을("example") 사용하였으므로 error 가 발생한다.

<br>
이제 `resource` block 안의 내용에 대해 살펴보자.  

### Arguments

`resource` block 안에는 해당 resource 를 생성하기 위한 정보인 arguments 가 들어간다.  
예제를 보면 `name = "example"` 이렇게 선언이 되어있는데 IAM user 의 이름을 "example" 로 하겠다는 뜻이다.  
Arguments 에 대한 정보는 어디서 얻을 수 있을까?  
물론 사용하는 provider 의 documentation 페이지에서 해당 resource 를 선택하면 볼 수 있다.  
예제에서 사용한 ["aws_iam_user" 의 documentation page](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_user){:target="_blank"} 로 이동해서 살펴보자.  
기본적으로 해당 resource 를 사용하는 example 이 나와있고, 조금 더 아래로 내려가보면 **`Argument Reference`** 으로 사용가능한 argument list 가 나와있다. (편의를 위해 이미지를 첨부한다.)

![Argument Reference]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-resource/resource-argument-reference.png)

List 를 보면 사용할 수 있는 모든 arguments 가 나오고 각각의 argument 옆에는 *(Required)* 또는 *(Optional)* 로 표시가 되어있다.  
써있는 그대로 *Required* 는 resource 를 생성하기 위해 반드시 명시되어야 하는 argument 를 뜻하고, *Optional* 은 추가적으로 resource 의 세부사항을 control 하고 싶을때 사용을 하면 된다.  
이것도 예시 코드와 함께 확인해 보겠다.

> **[Required, Optional 함께 사용]**
>
> ```terraform
> resource "aws_iam_user" "example" {
>   name = "example"
>   path = "/system/"
> }
> ```
>
> "example" 이란 IAM user 가 default path 인 "/" 가 아니라 "/system/" path 와 함께 생성된다.  
>
> **[Required 없이 사용, Error!]**
>
> ```terraform
> resource "aws_iam_user" "example" {
>   path = "/system/"
> }
> ```
>
> *Required* argument 를 사용하지 않았으므로 error 가 발생한다.

### Attributes

*Arguments* 가 resource 를 생성하기 위한 input 이라고 생각한다면, output 에 해당하는 *Attributes* 에 대해 알아보자.  
resource documentation 에 list 가 나와있으며, Arguments 와 동일하게 예제에서 사용한 ["aws_iam_user" 의 documentation page](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_user){:target="_blank"} 로 이동해서 살펴보자.  
**`Argument Reference`** 밑으로 내려가면 **`Attributes Reference`** 에 사용가능한 list 를 확인할 수 있다. (편의를 위해 이미지를 첨부한다.)  

![Argument Reference]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/terraform-resource/resource-attributes-reference.png)

이미지에서 확인 할 수 있듯이 `aws_iam_user` 는 3개의 *attributes* 를 가진다.  
*Attributes* 는 참조가 가능하기 때문에 필요한 곳 어디에서든지 사용이 가능하다.  
간단하게 다른 resource 에서 참조하는 예제를 통해 확인해 보자.  

<script src="https://gist.github.com/gentledev10/f5c8f0e6cd5f14ea1fc5f4e17633f4a1.js"></script>

`aws_s3_bucket` resource 의 bucket name 을 선언하는 곳을 보면 attributes 를 참조하는 방법에 대해 확인할 수 있다.  
`<RESOURCE TYPE>.<NAME>.<ATTRIBUTE>` 이 *attributes* 를 참조하는 문법이며, 위의 예제에서는 `aws_iam_user` resource type 의 `example` 이름을 가진 resource 를 선택해서 `name` *attribute* 를 참조하고 있다.  
위의 예제의 코드로 `terraform apply` 를 해보면 생성되는 s3 의 bucket 이름은 iam user 의 이름인 "example" 로 생성됨을 확인할 수 있다.  
<br>
<br>
지금까지 **resource** 를 code 로 작성하는 방법에 대해 알아보았다.
다음 포스트에서는 resource 2탄으로 **Meta-Arguments** 에 대해 알아보겠다.
