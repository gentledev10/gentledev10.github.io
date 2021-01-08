---
title: "Terraform install 및 명령어"
excerpt: "Terraform을 install 하고 명령어를 사용해보자"
categories:
  - Terraform
tags:
  - Terraform
  - Devops
---

# Install

먼저 **Terraform** 을 사용하기 위해 Install 을 하자.  
Install 은 굉장히 간단하다.  
<https://www.terraform.io/downloads.html>{:target="_blank"} 사이트를 방문하여 해당하는 OS의 Terraform 을 받으면 끝이다.  
혹시 Terraform 을 사이트에서 다운받지않고 package manager 를 통해 설치하고자 한다면 OS 에 따라 아래와 같이 설치가 가능하다.

>**[MacOS]**
>```bash
>$ brew install terraform
>```
>또는 Hashicorp repository 를 추가하여 설치하여도 동일하다.
>```bash
>$ brew tap hashicorp/tap
>$ brew install hashicorp/tap/terraform
>```

>**[Ubuntu/Debian]**
>```bash
>$ curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
>$ sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
>$ sudo apt-get update && sudo apt-get install terraform
>```

Download 또는 Install이 정상적으로 되었는지는 아래의 방법으로 쉽게 확인이 가능하다.
```bash
$ terraform version
Terraform v0.13.5 # 버전이 정상적으로 나타나면 성공!
```
<br>
# 명령어

Install 이 끝났으니, Terraform 에서 자주 사용하는 몇가지 명령어에 대해 알아보겠다.

## terraform init

Terraform을 사용하기 위해, 설정한 provider 나 module 등을 download 받는 initialize 동작을 수행한다.  
provider 나 module 은 앞으로 올릴 post 에서 설명을 할텐데, 우선은 library라고 생각하면 될 것 같다.  
Test를 위해 provider 를 AWS로 설정해주는 아래의 `tf` 파일을 생성해보자.  
(Code의 access_key 와 secret_key 는 직접 사용하는 AWS iam user 의 정보를 입력해야한다.)
<script src="https://gist.github.com/gentledev10/911d9323465d2be7977f52ef47a4b003.js"></script>
<br>
`tf` 파일을 생성했다면 파일이 있는 곳에서 `terraform init` 명령어를 실행해보자.  

```bash
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "3.14.1"...
- Installing hashicorp/aws v3.14.1...
- Installed hashicorp/aws v3.14.1 (signed by HashiCorp)

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

AWS provider 가 download 가 되고, 해당 폴더의 `.terraform` 폴더 밑에 저장되어 있는 걸 확인할 수 있다.  
:information_source: `terraform init`은 provider 나 module 등의 initialization 정보가 바뀔때만 한번씩 실행해 주면 된다.  

## terraform plan

`terraform plan` 명령어는 내가 code 로 작성한 infrastructure 를 실제로 적용하기 전에 변경사항을 체크해볼 수 있는 명령어다.  
확인을 위해 위의 `terraform init`을 위해 생성한 `tf` 파일에 아래의 내용을 포함 시킨다.  
(example 이라는 이름의 IAM User 를 만들어 주는 코드이다.)
<script src="https://gist.github.com/gentledev10/a5dbbcaff63e36de29d23789ba49a3c6.js"></script>
<br>
자 이제 `terraform plan` 명령어를 실행해 보자.  

```bash
$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_iam_user.example will be created
  + resource "aws_iam_user" "example" {
      + arn           = (known after apply)
      + force_destroy = false
      + id            = (known after apply)
      + name          = "example"
      + path          = "/"
      + unique_id     = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

`+` 표시와 함께 생성되는 resource 가 무엇인지 보여주는 화면이 나타난다.  
`terraform plan`을 통해 나타나는 resource 의 변경 결과는 `create`, `update`, `destroy` 중 하나다.  

- `+` -> create
- `~` -> update
- `-` -> destroy

## terraform apply

`terraform apply` 명령어는 code 로 작성된 infrastructure 를 실제로 적용시키는 명령어다.  
위에서 생성한 `tf` 파일을 그대로 사용하여 `terraform apply` 명령어를 실행해보자.

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
      + name          = "example"
      + path          = "/"
      + unique_id     = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

위에서 사용해봤던 `terraform plan`과 동일한 결과를 보여주며 적용을 하려면 `yes`를 입력하라고 나온다.  
적용을 위해 `yes`를 입력해보자.  

```bash
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_iam_user.example: Creating...
aws_iam_user.example: Creation complete after 3s [id=example]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

`Creating...` 이란 문구가 나오면서 잠시후에 `Creation complete` 문구를 확인할 수 있다.  
Terraform 을 통해, 우리가 작성한 code 를 실제 infrastructure 에 적용을 완료했다!  
AWS console 에 IAM -> Users 에 들어가보면 `example` 이라는 user 가 생성된 걸 알 수 있다.  

![aws-iam-user]({{ site.url }}{{ site.baseurl }}/assets/images/terraform/install-terraform-and-commands/new-iam-user.png)

## terraform destroy

`terraform destroy` 명령어는 terraform 으로 생성된 insrastructure resource 들을 모두 제거하고 싶을 때 사용하는 명령어다.  
앞에서 `terraform apply` 를 통해 AWS IAM user 를 생성했던 걸 지워보자.  
동일한 위치에서 `terraform destroy` 명령어를 실행해보자.

```bash
$ terraform destroy
aws_iam_user.example: Refreshing state... [id=example]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_iam_user.example will be destroyed
  - resource "aws_iam_user" "example" {
      - arn           = "arn:aws:iam::801399456697:user/example" -> null
      - force_destroy = false -> null
      - id            = "example" -> null
      - name          = "example" -> null
      - path          = "/" -> null
      - tags          = {} -> null
      - unique_id     = "AIDA3VFZBIO4VXK234GXC" -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value:
```

위에서 AWS IAM user 를 한개 만들어놨기 때문에 해당 resource 가 삭제된다는 걸 알려주는 화면이 표시된다.  
`yes` 를 입력해서 해당 resource 를 삭제해 보자.

```bash
Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_iam_user.example: Destroying... [id=example]
aws_iam_user.example: Destruction complete after 2s

Destroy complete! Resources: 1 destroyed.
```

`Destroying...` 이라는 문구가 나오면서 잠시후에 `Destruction complete` 문구와 함께 우리가 만들었던 resource 가 삭제된걸 확인할 수 있다.  
AWS console 에 들어가서 확인해봐도 해당 resource 가 삭제된 걸 확인할 수 있다.  
<br>
<br>
지금까지 Terraform 의 install 방법과 주요 command 에 대해 살펴봤다.  
물론 Terraform 은 더 많은 command 들을 사용할 수 있지만, 오늘 살펴본 4개의 command 만으로도 Terraform 으로 infrastructure 를 구축하는데 문제가 없기에 4가지의 command 만 살펴보았다.  
혹시 더 많은 command 가 궁금하다면, <https://www.terraform.io/docs/commands/index.html>{:target="_blank"} 여기를 확인해 보기 바란다.  
추후 더 많은 command 를 다루는 post 를 올려보도록 하겠다.  
