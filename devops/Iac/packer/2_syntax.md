# Packer - Syntax

**Ref**

* [syntax](https://www.packer.io/docs/templates/hcl_templates/blocks)

## Blocks

* locals : 지역변수
* variable : 입력 변수
* packer(=terraform) : 의존성과 버전 설정
* data : 데이터 소스
* source : 이미지 소스
* build : 이미지 빌드

### locals

```json
# locals.pkr.hcl
locals {
    # locals can be bare values like:
    wee = local.baz
    # locals can also be set with other variables :
    baz = "Foo is '${var.foo}' but not '${local.wee}'"
}
```

여러개의 지역 변수를 선언

### local

```json
# Use the singular local block if you need to mark a local as sensitive
local "mylocal" {
  expression = "${var.secret_api_key}"
  sensitive  = true
}
```

한 개의 지역 변수를 선언한다.

* expression : value 값
* sensitive : environment 변수인지 아닌지 설정
