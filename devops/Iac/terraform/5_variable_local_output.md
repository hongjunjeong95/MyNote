# Terraform - Variable & Local & Output

**Ref**

* https://www.terraform.io/docs/language/values/index.html

## Variable

### Usage

```json
variable "vpc_name" {}
```

* Block type으로 `variable`을 사용
* variable을 참조할 때는 `var.vpc_name`으로 `var.<label_name>`을 사용

#### Variable definition precedence

1. Any `-var` and `-var-file` options on the command line, in the order they are provided. (This includes variables set by a Terraform Cloud workspace.)
2. Any `*.auto.tfvars` or `*.auto.tfvars.json` files, processed in lexical order of their filenames.
3. The `terraform.tfvars.json` file, if present.
4. The `terraform.tfvars` file, if present.
5. Environment variables

#### -var & -var-file

```shell
tf apply -var-file=_terraform.auto.tfvars
tf apply -var="vpc_name=fastcampus"
```

위 명령어로 변수 파일을 지정하고 변수를 선언할 수 있다.

#### *.auto.tfvars

만약 `terraform.tfvars` 파일 이름 이외에 다른 이름으로 파일을 생성해서 변수를 관리하고 싶다면, 미들 네임으로 `auto`를 추가한다.

```yaml
# _terraform.auto.tfvars

vpc_name="fastcampus"
```

#### terraform.tfvars

```json
vpc_name="fastcampus"
```

위 파일로 변수 관리 가능

#### Environment variables

`export TF_VAR_vpc_name="test"`

환경 변수를 등록하고 싶으면 `TF_VAR`를 앞에 붙인다. `vpc_name="test"`가 변수가 된다.

### Arguments

* [`default`](https://www.terraform.io/docs/language/values/variables.html#default-values)(optional) - A default value which then makes the variable optional.
* [`type`](https://www.terraform.io/docs/language/values/variables.html#type-constraints)(optional) - This argument specifies what value types are accepted for the variable.
* [`description`](https://www.terraform.io/docs/language/values/variables.html#input-variable-documentation)(optional) - This specifies the input variable's documentation.



## Local

### Usage

```json
locals {
  common_tags = {
    Project = "Network"
    Owner   = "hongjun"
  }
}
```



## Output

### Usage

```json
output "vpc_name" {
  value = module.vpc.name
}

output "vpc_id" {
  value = module.vpc.id
}

output "vpc_cidr" {
  description = "생성된 VPC의 CIDR 영역"
  value = module.vpc.cidr_block
}

output "subnet_groups" {
  value = {
    public  = module.subnet_group__public
    private = module.subnet_group__private
  }
}
```

다른 모듈이 해당 값을 참조하기 위해서 output으로 선언해줘야 함

### Arguments

* value(required)
* description(optional)
* sensitive(optional)
* depends_on(optional)