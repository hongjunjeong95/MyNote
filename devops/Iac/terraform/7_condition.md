# Terraform - Condition

**Ref**

* condition : https://www.terraform.io/docs/language/expressions/conditionals.html

```json
provider "aws" {
  region = "ap-northeast-2"
}


/*
 * Conditional Expression
 * Condtion ? If_True : If_False
 */
variable "is_john" {
  type = bool
  default = true
}

locals {
  message = var.is_john ? "Hello John!" : "Hello!"
}

output "message" {
  value = local.message
}
```

3항 연산자를 이용해서 조건문을 사용한다.

```json
/*
 * Count Trick for Conditional Resource
 */
variable "internet_gateway_enabled" {
  type = bool
  default = true
}

resource "aws_vpc" "this" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_internet_gateway" "this" {
  count = var.internet_gateway_enabled ? 1 : 0

  vpc_id = aws_vpc.this.id
}
```

count와 3항 연산자를 이용해서 resource를 생성하거나 생성하지 않을 수 있다. `internet_gateway_enabled`가 `true`면 IGW를 생성하거 `false`면 IGW를 생성하지 않는다.