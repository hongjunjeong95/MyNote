# Terraform - for

**Ref**

* for : https://www.terraform.io/docs/language/expressions/for.html

테라폼에서 expression을 사용할 수 있는 모든 곳에서 사용할 수 있다.

## Example

### main.tf

```json
provider "aws" {
  region = "ap-northeast-2"
}

/*
 * Groups
 */

resource "aws_iam_group" "developer" {
  name = "developer"
}

resource "aws_iam_group" "employee" {
  name = "employee"
}

output "groups" {
  value = [
    aws_iam_group.developer,
    aws_iam_group.employee,
  ]
}


/*
 * Users
 */

variable "users" {
  type = list(any)
}

resource "aws_iam_user" "this" {
  for_each = {
    for user in var.users :
    user.name => user
  }

  name = each.key

  tags = {
    level = each.value.level
    role  = each.value.role
  }
}

resource "aws_iam_user_group_membership" "this" {
  for_each = {
  // user.name은 key가 되고
  // user는 value가 된다.
    for user in var.users :
    user.name => user
  }
// each.key == user.name
// each.value == user
  user   = each.key
  groups = each.value.is_developer ? [aws_iam_group.developer.name, aws_iam_group.employee.name] : [aws_iam_group.employee.name]
}

locals {
  developers = [
  // user.is_developer가 true인 경우 user를 반환
    for user in var.users :
    user
    if user.is_developer
  ]
}

resource "aws_iam_user_policy_attachment" "developer" {
  for_each = {
    for user in local.developers :
    user.name => user
  }

  user       = each.key
  policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess"

  // 의존성 관리 속성이다.
  // aws iam user가 생성되었을 때 정책을 해당 유저에게 attach하는 리소스를 생성한다.
  depends_on = [
    aws_iam_user.this
  ]
}

output "developers" {
  value = local.developers
}

output "high_level_users" {
  value = [
  // user.level > 5 일 때, user를 반환
    for user in var.users :
    user
    if user.level > 5
  ]
}
```

### terraform.tfvars

```json
users = [
  {
    name = "john"
    level = 7
    role = "재무"
    is_developer = false
  },
  {
    name = "alice"
    level = 1
    role = "인턴 개발자"
    is_developer = true
  },
  {
    name = "tony"
    level = 4
    role = "데브옵스"
    is_developer = true
  },
  {
    name = "cindy"
    level = 9
    role = "경영"
    is_developer = false
  },
  {
    name = "hoon"
    level = 3
    role = "마케팅"
    is_developer = false
  },
]
```
