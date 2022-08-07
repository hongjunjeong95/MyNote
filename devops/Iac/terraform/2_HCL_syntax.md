# Terraform - HCL Syntax

**Ref**

* https://www.terraform.io/docs/language/index.html

## Basic syntax

```json
resource "aws_vpc" "main" {
  cidr_block = var.base_cidr_block
}

<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```

* Label에 들어올 수 있는 것은 letters, digits, underscores(_) and hyphens(-)이다. 숫자는 첫번째 문자로 올 수 없다.

## Text Encoding

* UTF-8 encoding을 반드시 사용
* 개행 문자로 Unix-style인 LF 사용

## Directories and Modules

```shell
tf init
tf plan
tf apply
```

위 명령어를 쳤을 때 terraform은 하위 디렉토리를 탐색하지 않는다.

## Style Conventions

* Indent two spaces for each nesting level
  * tab X
    * Don't use "tab"
* meta-argument
  * 앞에 위치 : count, for_each etc...
  * 뒤에 위치 : lifecycle, depends_on etc...
* `tf fmt` 명령어로 포맷팅 가능
  * `tf fmt -diff`로 어떤 변경 사항이 있는지 확인 가능
