# Terraform - Module

**Ref**

* [Terraform module](https://www.terraform.io/language/modules)

## 모듈 구조

![image](https://user-images.githubusercontent.com/92770273/146290482-4bb4d62f-8b18-40e2-9d02-cdeeac277550.png)

Accont 폴더는 모듈이다. 각각의 파일명은 컨벤션이며 다음의 항목을 정의한다.

* outputs.tf : output
* variables.tf : variable 변수
  * default로 변수 할당
  * terraform.tfvars를 통해서 변수 할당
  * variables.tf가 모듈로 root module에게 사용된다면 root에서 account 모듈을 사용할 때 변수 할당할 수 있다.
* versions.tf : provider / module 버전 의존성
* main.tf : resource, data, module, local
  * main.tf에서는 resource가 주로 정의되며 나머지는 또 다른 파일로 나눌 수 있다.
* data.tf : data
* local.tf : local 변수
* provider.tf : provider
* backend.tf : state storage 정의
* remote.tf : remote storage에서 가져올 리소스 주소 정의
* terraform.tfvars : variables.tf에 할당될 변수
* README.md : 모듈에 대한 가이드 문서

## 모듈을 이용한 변수 할당

```json
// account/variables.tf

variable "name" {
  description = "The name for the AWS account. Used for the account alias."
  type        = string
}

variable "password_policy" {
  description = "Password Policy for the AWS account."
  type = object({
    minimum_password_length        = number
    require_numbers                = bool
    require_symbols                = bool
    require_lowercase_characters   = bool
    require_uppercase_characters   = bool
    allow_users_to_change_password = bool
    hard_expiry                    = bool
    max_password_age               = number
    password_reuse_prevention      = number
  })
  default = {
    minimum_password_length        = 8
    require_numbers                = true
    require_symbols                = true
    require_lowercase_characters   = true
    require_uppercase_characters   = true
    allow_users_to_change_password = true
    hard_expiry                    = false
    max_password_age               = 0
    password_reuse_prevention      = 0
  }
}
```

```json
// main.tf
provider "aws" {
  region = "ap-northeast-2"
}

module "account" {
  source = "./account"

  name = "posquit0-fastcampus"
  password_policy = {
    minimum_password_length        = 8
    require_numbers                = true
    require_symbols                = true
    require_lowercase_characters   = true
    require_uppercase_characters   = true
    allow_users_to_change_password = true
    hard_expiry                    = false
    max_password_age               = 0
    password_reuse_prevention      = 0
  }
}
```

`main.tf`에서 account 모듈을 가져오고 name과 password_policy 변수에 값을 할당

## 모듈 속성 참조

```json
// main.tf

provider "aws" {
  region = "ap-northeast-2"
}

module "account" {
  source = "./account"

  name = "posquit0-fastcampus"
  password_policy = {
    minimum_password_length        = 8
    require_numbers                = true
    require_symbols                = true
    require_lowercase_characters   = true
    require_uppercase_characters   = true
    allow_users_to_change_password = true
    hard_expiry                    = false
    max_password_age               = 0
    password_reuse_prevention      = 0
  }
}

output "id" {
  value = module.account.id
}

output "account_name" {
  value = module.account.name
}

output "signin_url" {
  value = module.account.signin_url
}

output "account_password_policy" {
  value = module.account.password_policy
}
```

```json
// account/outputs.tf

output "id" {
  description = "The AWS Account ID."
  value       = data.aws_caller_identity.this.account_id
}

output "name" {
  description = "Name of the AWS account. The account alias."
  value       = aws_iam_account_alias.this.account_alias
}

output "signin_url" {
  description = "The URL to signin for the AWS account."
  value       = "https://${var.name}.signin.aws.amazon.com/console"
}

output "password_policy" {
  description = "Password Policy for the AWS Account. `expire_passwords` indicates whether passwords in the account expire. Returns `true` if `max_password_age` contains a value greater than 0."
  value       = aws_iam_account_password_policy.this
}
```

`module.account.*`로  해당 모듈의 속성을 참조하고 싶다면 해당 모듈에서 output으로 내보내줘야 한다.

## Terraform docs

**Ref**

* [Terraform docs](https://github.com/terraform-docs/terraform-docs)

다양한 포맷으로 테라폼 모듈에 대한 문서를 만들어준다.

* pre-commit hook : git을 통해서 commit을 하기 전에 자동으로 hook으로 동작해서 commit 전에 여러 가지 도구들이 시작될 수 있도록 도와준다. 사용 예제로는 커밋전에 terraform docs를 실행시켜서 문서를 자동으로 최신화시킬 수 있다.
