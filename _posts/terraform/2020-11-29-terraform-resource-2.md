---
title: "Terraform resource (2/2)"
excerpt: "Resource 의 Meta-Arguments 에 대해 알아보자."
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Resource Meta-Arguments

이전 [포스트]({% post_url /terraform/2020-11-21-terraform-resource-1 %}) 에서 `resource` block 안에서 사용할 수 있는 *Arguments* 란 무엇인지 알아 보았다.  
각각의 `resource` 들은 해당 `resource` 를 생성하기 위해 각기 다른 *Arguments* 를 선언할 수 있다.  
하지만 이렇게 각기 다른 *Arguments* 외에 모든 `resource` 가 공통적으로 사용 할 수 있는 *Arguments* 가 있는데, 이걸 ***Meta-Arguments*** 라고 부른다.  
이번 포스트에서는 이 ***Meta-Arguments*** 의 종류와 사용법에 대해서 알아보겠다.  

## 1. depends_on

`depends_on` 이란 이름 그대로 특정 `resource` 에 dependency 를 설정해 줄때 사용하는 arguments 이다.  
쉽게 생각하면 dependency 를 설정함으로써 `resource` 들의 실행 순서를 정해줄 수 있다.  
더 쉬운 이해를 위해 예제를 통해 설명하겠다.

<script src="https://gist.github.com/gentledev10/8d63296b21ba8110ee2134d20e227d91.js"></script>

먼저 위의 코드를 설명하기 전, 개발자에게 다음과 같은 상황이 주어졌다고 가정하자.  
> S3 bucket 하나와 EC2 instance 하나를 생성해야 하고, EC2 instance 생성 시 S3 에 파일을 push 하는 작업을 수행해야 한다.  

위의 코드는 이 요구사항을 만족시키는 코드이다.  
하나씩 살펴보자.  

먼저 `aws_s3_bucket` resource 를 통해 s3 bucket 을 생성한다.  
그리고 `aws_instance` resource 를 통해 EC2 를 생성하고, `user_data` argument 를 통해 EC2 생성 시 S3 에 파일을 push 하는 작업을 수행하도록 한다.  
그렇다면 여기에서 가장 중요하게 고려해야 할 상황은 EC2 가 생성되기 전에 S3 bucket 이 생성 완료되야 파일을 push 하는 작업을 문제 없이 수행할 수 있다는 점이다.  
이 문제의 해결을 위해 `depends_on` 이 사용됐다!  

`depends_on` block 안에 `aws_s3_bucket.example` 을 선언함으로써 `aws_instance.example` resource 는 `aws_s3_bucket.example` resource 에 dependency 를 가지고 있다고 terraform 에게 명시적으로 알려주게 된다.  
따라서 `aws_s3_bucket.example` resource 가 생성이 완료 된 후에 `aws_instance.example` resource 가 생성을 시작하게 된다.  

:interrobang: 그렇다면 모든 terraform resource 를 생성할때 dependency 를 고려하여 `depends_on` 을 추가해야 하는걸까?  
**절대 아니다!** ~~생각만 해도 끔찍하다~~  
다음의 예제를 살펴보자.  

<script src="https://gist.github.com/gentledev10/5d657f374f28ca016fb2e5ccdb53cf96.js"></script>

`aws_s3_bucket` 의 bucket 이름을 `aws_iam_user` 의 arn 으로 설정하도록 되어 있는 terraform 코드이다.  
`aws_s3_bucket` 에서 `aws_iam_user.example.arn` 을 사용하고 있기때문에 terraform 은 자동으로 `aws_s3_bucket` 이 `aws_iam_user.example` 에 dependency 를 가지고 있음을 알 수 있게 되고, IAM user 가 먼저 생성된 후 생성된 arn 을 사용하여 S3 bucket 을 생성하게 된다.  

결국 `depends_on` 은 resource 간의 dependency 설정이 필요한데 terraform 이 자동으로 dependency 를 알기 어려운 경우에만 사용해주면 된다!  
한가지 더 얘기하자면 `depends_on` 을 사용시에는 추후 관리를 위해 왜 이것이 필요한지 comment 를 함께 작성해주면 도움이 된다.  

## 2. count

일반적으로 `resource` block 을 통해 resource 를 생성하면 1개의 resource 가 생성이 되는데, 동일한 `resource` block 으로 여러개의 동일한 resource type 을 생성하고 싶을때 사용하는 argument 가 `count` 이다.  
바로 예제를 통해 살펴보자.  

<script src="https://gist.github.com/gentledev10/846cd53d5ac1e2942cda1e3b61573fba.js"></script>

`aws_iam_user` 를 생성하는 `resource` block 에 `count` argument 를 추가하여 3개의 IAM user 를 생성해주는 코드이다.  
사용법은 이렇게 상당히 간단하며, 위의 code 를 apply 해보면 각각 *example_0*, *example_1*, *example_2* 이름을 가진 3개의 AWS iam user 가 생성되게 된다.

### count object

`count` argument 를 사용하면 특별히 `count` object 를 `resource` block 안에서 사용할 수 있게 된다.  
이 `count` object 를 통해서 생성되는 resource 의 index 값을 가져올 수 있는데, 이때 사용하는 방법이 `count.index` 이다.  
`count.index` 의 값은 0부터 시작하고 해당 `resource` block 내의 어디서든 사용이 가능하다.  

### resource instance 참조

기존에 우리는 생성된 resource 를 참조하기 위해 `<RESOURCE TYPE>.<NAME>` 문법을 사용하여 `aws_iam_user.example` 이런식으로 사용을 했다.  
하지만 `count` argument 를 사용하여 생성한 resource 를 참조하기 위해서는 `<TYPE>.<NAME>[<INDEX>]` 문법을 사용하여 resource 의 index 를 명시해 줘야 한다.  
`aws_iam_user.example[0]` 이런 식으로 참조를 할 수 있으며 예를 들어 이름을 참조하고 싶다면 `aws_iam_user.example[0].name`, `aws_iam_user.example[1].name` 등등 만들어진 count 에 따라 index 를 함께 명시함으로써 참조가 가능하다.  

### 주의

`count` argument 의 값은 apply 전에 정해져 있어야 한다.  
`count = aws_iam_user.example.name == "" : 0 : 1` 이렇게 count 값에 apply 후에 가져올 수 있는 resource 의 attribute 를 참조하는 코드가 있다거나 하면 terraform 에서는 error 를 발생시킨다.  

## 3. for_each

terraform 0.12.6 버전 이후 사용할 수 있는 meta-argument 로 `count` 와 동일하게 한개의 `resource` block 으로 여러개의 동일한 resource type 을 생성하고자 할때 사용한다.  
`count` 와 `for_each` 는 `resource` block 에서 동시에 사용할 수 없으므로 한개만 선택해서 사용하도록 한다.  
`for_each` 는 map 또는 set 을 값으로 가질 수 있으며 map 또는 set 을 통해 전달된 값의 갯수만큼 resource 를 생성한다.  
예제코드를 통해 살펴보자.  

<script src="https://gist.github.com/gentledev10/4dbcc663e608ec497f429e845678fca0.js"></script>

먼저 첫번째 `example1` resource 는 set 을 사용하여 *user1*, *user2*, *user3* 총 3개의 IAM user 를 만드는 코드이다.  
두번째 `example2` resource 는 map 을 사용하여 *user4*, *user5*, *user6* 총 3개의 IAM user 를 각각 *tag4*, *tag5*, *tag6* 태그와 함께 생성하는 코드이다.  

### each object

`for_each` 를 사용하여 resource 를 생성하면, `each` object 를 `resource` block 내에서 사용할 수 있다.  
`each` object 를 통해 `each.key`, `each.value` 를 사용할 수 있다.  
`each.key` 는 map 을 사용시에는 key 값을, set 을 사용시에는 member 값을 의미한다.  
`each.value` 는 map 을 사용사에는 value 값을, set 을 사용시에는 `each.key` 와 동일하게 member 값을 의미한다.  

### resource instance 참조

`for_each` 를 사용하여 생성한 resource 를 참조하기 위해서는 `<TYPE>.<NAME>[<KEY>]` 문법을 사용한다.  
위의 예제 코드에서 set 을 사용한 resource 참조를 위해서는 `aws_iam_user.example1["user2"].name`, `aws_iam_user.example1["user3"].name` 이런식으로 사용하면 되고,  
map 을 사용한 resource 참조도 동일하게 `aws_iam_user.example1["user4"].name`, `aws_iam_user.example1["user5"].name` 이런식으로 가능하다.  

### 주의

`count` argument 와 마찬가지로 `for_each` 의 값도 apply 전에 정해져 있어야 한다.  
그렇지 않으면 apply 시 error 가 발생하니 주의하기 바란다.  

## 4. provider

AWS infrastructure 를 구성하다보면 종종 다른 configuration 을 가지고 (예를 들면 다른 region) resource 를 생성해야 하는 경우가 생긴다.  
이럴때 사용할 수 있는 meta-argument 가 바로 `provider` 이다.  
예제 코드롤 통해 더 자세히 살펴보자.  

<script src="https://gist.github.com/gentledev10/1a0800bb042954717ca6ecc93b37b4dd.js"></script>

먼저 aws provider 를 선언해준 부분을 살펴보자.  
`us-east-1` region 을 사용하는 default configuration 한개와 `ap-northeast-2` region 을 사용하는 alternate configuration 한개가 있다.  
provider 당 default configuration 은 반드시 1개만 선언할 수 있기 때문에 `ap-northeast-2` region 을 사용하는 configuration 은 `alias` 를 선언하여 alternate 로 만들어 주었다. (`alias`에 대해서는 [공식문서](https://www.terraform.io/docs/configuration/providers.html#alias-multiple-provider-configurations){:target="_blank"}에도 잘 나와있다.)  

이렇게 aws provider 에 대해 2개의 configuration 을 사용할 수 있는 환경이 만들어졌고, `resource` block 에서 `provider` 를 선언하지 않고 default configuration 을 사용하던가 `provider` 를 선언하여 원하는 configuration 을 선택할 수가 있다.  

예제 코드에서 `aws_instance.us-east-1` resource 는 `provider` 를 선언하지 않았기 때문에 default configuration 을 사용하여 `us-east-1` region 에 EC2 instance 를 생성하고, `aws_instance.ap-northeast-2` resource 는 `provider = aws.seoul` 을 선언하여 `seoul` 이라는 `alias` 를 가지는 alternate configuration 을 사용하여 `ap-northeast-2` region 에 EC2 instance 를 생성하게 된다.  

## 5. lifecycle

`lifecycle` meta-argument 는 resource 의 create, update, destroy 에 관여하여 해당 동작의 수행을 우리가 원하는 방식으로 변경할때 사용하는 argument 이다.  
`lifecycle` 내에서 사용할 수 있는 argument 에는 총 3가지가 있으며 예제와 함께 살펴보자.  

<script src="https://gist.github.com/gentledev10/ff30cfae1f3af458acef02f5be3c5959.js"></script>

### create_before_destroy

Terraform 에서는 특정 resource 에 대해 update 를 해야하나 제약사항에 의해 update 가 불가능한 경우 만들어진 resource 를 삭제하고 update 된 resource 를 새로 만드는 것이 기본 동작이다.  
`create_before_destroy` argument 는 bool 값을 가지며 `true` 로 설정하면 이런 경우 먼저 update 된 resource 를 생성하고 기존의 resource 를 삭제하는 방식으로 동작하도록 할 수 있다.  
단 이 동작을 수행하고자 하는 resource 의 type 에 따라 unique constraint 등 다른 제약들이 있어 불가능한 경우가 있으니, 사용전 확인해보고 사용을 하는걸 추천한다.  

### prevent_destroy

Terraform 을 사용하여 생성된 resource 들 중 destroy 가 되는걸 방지하고 할때 사용하는 argument 이다.  
`prevent_destroy` 는 bool 값을 가지며 `true` 로 설정하면 resource 가 destroy 되려고 할때 error 를 던진다. (`terraform destroy` 시에도 적용이 된다.)  

### ignore_changes

Terraform 을 사용하여 infrastructure 구축 시 terraform 은 실제 적용되어 있는 resource 들의 값과 code 로 작성되어 적용하고자 하는 값들을 비교하여 해당 resource 의 create, update, destroy 를 결정한다.  
만약 특정 resource 가 terraform 으로도 관리되고, console 이나 다른 곳에서도 함께 관리를 하고 있다고 생각해보자.(물론 이렇게 관리되는건 추천하지 않는다. Terraform 을 사용한다면 Terraform 을 통해서만 관리하는게 가장 좋은 방법이다.)  
누군가가 console 을 통해 resource 의 어떤 값을 바꿧다면 terraform 수행시에 terraform 은 해당 값이 변경된 것을 확인하고 code 에 있는 값으로 다시 원복 시키려 할 것이다.  
이런 불상사를 방지하기 위해 사용되는 argument 가 `ignore_changes` 이다. (물론 다른 여러 이유로도 사용이 필요한 경우가 있을 것이다.)  
`ignore_changes` 는 list 값을 가지며 list 에 적은 arguments 를 terraform 이 비교하는 대상에서 제외시켜 update 를 하지 않는다.  
비교 대상에서 제외하고자 하는 값을 list 안에 넣어주면 되고, 혹시 모든 arguments 를 비교대상에서 제외하고자 한다면 `example4` 의 예처럼 list 대신 `all` 값을 선언해주면 된다.  

<br>
<br>
지금까지 ***Meta-arguments*** 에 대해 살펴보았다.  
***Meta-argunemts*** 는 모든 `resource` block 에서 사용이 가능하다는 걸 기억하고, 필요한 곳에 사용하여 더 안정적인 resource 운영에 도움이 되길 바란다!  

<br>
<br>
**[참고]**  

<https://www.terraform.io/docs/configuration/meta-arguments/depends_on.html>{:target="_blank"}  
<https://www.terraform.io/docs/configuration/meta-arguments/count.html>{:target="_blank"}  
<https://www.terraform.io/docs/configuration/meta-arguments/for_each.html>{:target="_blank"}  
<https://www.terraform.io/docs/configuration/meta-arguments/resource-provider.html>{:target="_blank"}  
<https://www.terraform.io/docs/configuration/meta-arguments/lifecycle.html>{:target="_blank"}
