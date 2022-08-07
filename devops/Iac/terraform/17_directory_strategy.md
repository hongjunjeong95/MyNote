# Terraform - Directory Strategy

<img src="https://user-images.githubusercontent.com/92770273/146303456-fdb4cdfe-c3b6-4db5-91c7-68e5c8fac1f2.png" alt="image" style="zoom:50%;" />

위 빨간색 박스를 친 부분이 중요하다. 해당 부분이 워크스페이스에서 공통적인 부분이고 해당 파일을 가지고 리소스를 생성하고 관리한다. 나머지 것(eip, nacl과 nat 등등)들은 리소스를 뜻한다.

## versions.tf

```json
terraform {
  required_version = "~> 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}
```

* required_version : 이 워크스페이스를 실행하기 위해 필요한 테라폼 최소 버전
* required_providers : provider의 목록과 버전

## variables.tf

```json
variable "config_file" {
  description = "The path of configuration YAML file."
  type        = string
  default     = "./config.yaml"
}
```

`config.yaml` 파일로 variable을 관리한다.

## terraform.auto.tfvars

```json
config_file = "./config.yaml"
```

파일 경로 설정

## config.yaml

```yaml
context:
  region: "apne2"
  vpc: "apne2-fastcampus"
  cidrs:
    "primary": "10.222.0.0/16"
    "pod": "10.223.0.0/17"

remote_states:
  "domain-zone":
    organization: "fastcampus-devops-lewisjeong"
    workspace: "aws-domain-zone"

prefix_lists:
  ipv4: []

vpc:
  name: "${vpc}"
  cidr: "${cidrs.primary}"
  secondary_cidrs:
    # EKS Pod CIDR
    - "${cidrs.pod}"
  vpn_gateway_asn: 4242000000
  private_zones:
    - "dev/dkr.ecr.ap-northeast-2.amazonaws.com"
    - "dev/sts.ap-northeast-2.amazonaws.com"

vpc_endpoints:
  interface: []
  gateway:
    - name: "${vpc}-gateway-aws-s3"
      service_name: "com.amazonaws.ap-northeast-2.s3"

subnet_groups:
  "app-private":
    subnets:
      - { cidr: "10.222.0.0/24", az_id: "${region}-az1" }
      - { cidr: "10.222.1.0/24", az_id: "${region}-az2" }
      - { cidr: "10.222.2.0/24", az_id: "${region}-az3" }
  "app-private-pod":
    subnets:
      - { cidr: "10.223.0.0/19", az_id: "${region}-az1" }
      - { cidr: "10.223.32.0/19", az_id: "${region}-az2" }
      - { cidr: "10.223.64.0/19", az_id: "${region}-az3" }
  "data-private-managed":
    subnets:
      - { cidr: "10.222.100.0/24", az_id: "${region}-az1" }
      - { cidr: "10.222.101.0/24", az_id: "${region}-az2" }
      - { cidr: "10.222.102.0/24", az_id: "${region}-az3" }
    db_subnet_group_name: "${vpc}-db-private"
    cache_subnet_group_name: "${vpc}-cache-private"
  "data-private-self":
    subnets:
      - { cidr: "10.222.103.0/24", az_id: "${region}-az1" }
      - { cidr: "10.222.104.0/24", az_id: "${region}-az2" }
      - { cidr: "10.222.105.0/24", az_id: "${region}-az3" }
  "net-public":
    map_public_ip_on_launch: true
    subnets:
      - { cidr: "10.222.230.0/24", az_id: "${region}-az1" }
      - { cidr: "10.222.231.0/24", az_id: "${region}-az2" }
      - { cidr: "10.222.232.0/24", az_id: "${region}-az3" }
  "net-private":
    subnets:
      - { cidr: "10.222.233.0/24", az_id: "${region}-az1" }
      - { cidr: "10.222.234.0/24", az_id: "${region}-az2" }
      - { cidr: "10.222.235.0/24", az_id: "${region}-az3" }
```



## terraform.tf

```json
terraform {
  backend "remote" {
    hostname     = "app.terraform.io"
    organization = "fastcampus-devops-lewisjeong"

    workspaces {
      name = "aws-network-apne2-fastcampus"
    }
  }
}


###################################################
# Local Variables
###################################################

locals {
  aws_accounts = {
    fastcampus = {
      id     = "xxxxxxxxxx"
      region = "ap-northeast-2"
      alias  = "posquit0-fastcampus"
    },
  }
  context = yamldecode(file(var.config_file)).context
  config  = yamldecode(templatefile(var.config_file, local.context))
}


###################################################
# Providers
###################################################

provider "aws" {
  region = local.aws_accounts.fastcampus.region

  # Only these AWS Account IDs may be operated on by this template
  allowed_account_ids = [local.aws_accounts.fastcampus.id]

  assume_role {
    role_arn     = "arn:aws:iam::${local.aws_accounts.fastcampus.id}:role/terraform-access"
    session_name = local.context.workspace
  }
}
```

* remote backend를 이용해서 state 저장소를 등록
* locals
  * aws account 등록
  * context를 등록하여 config_file에서 해당 변수(context)를 사용하도록 넘겨줌
    * file : 이 yaml config file 경로를 주면 해당 yaml 파일을 yamldecode가 yaml => hcl로 변환
    * templatefile : config file 경로와 config.yaml에서 사용할 context 변수를 인자로 받고 랜더링한다.
  * config 값이 terraform code에서 사용할 variables
* provider : provider 목록을 나열한다. 여기서는 aws provider만 사용했다.

## remote-states.tf

```json
locals {
  remote_states = {
    "domain-zone" = data.terraform_remote_state.this["domain-zone"].outputs
  }
  domain_zones = local.remote_states["domain-zone"]
}


###################################################
# Terraform Remote States (External Dependencies)
###################################################

data "terraform_remote_state" "this" {
  for_each = local.config.remote_states

  backend = "remote"

  config = {
    organization = each.value.organization
    workspaces = {
      name = each.value.workspace
    }
  }
}
```

Multi workspaces를 관리를 할 때 중요한 기능을 한다.

## outputs.tf

```json
output "config" {
  value = local.config
}

output "prefix_lists" {
  value = {
    ipv4 = {
      for name, prefix_list in aws_ec2_managed_prefix_list.ipv4 :
      name => {
        id      = prefix_list.id
        arn     = prefix_list.arn
        name    = prefix_list.name
        entries = prefix_list.entry
        version = prefix_list.version
      }
    }
    ipv6 = {
      for name, prefix_list in aws_ec2_managed_prefix_list.ipv6 :
      name => {
        id      = prefix_list.id
        arn     = prefix_list.arn
        name    = prefix_list.name
        entries = prefix_list.entry
        version = prefix_list.version
      }
    }
  }
}

output "vpc" {
  value = module.vpc
}

output "subnet_groups" {
  value = module.subnet_group
}

output "eip" {
  value = aws_eip.this
}

output "nat_gateway" {
  value = module.nat_gw
}

output "nacl" {
  value = module.nacl
}

output "gateway_endpoints" {
  value = module.vpc_gateway_endpoint
}
```

해당 워크 스페이스에서 사용자에게 보여줘야 하는 결과 값을 정의

## .terraform-version

```
1.0.0
```

테라폼 버전을 고정. 워크 스페이스를 관리하다 보면은 각각의 워크스페이스 별로 테라폼 버전이 다르게 될 수 있다. 작업자가 테라폼 워크 스페이스에 진입했을 때 자동으로 해당 워크스페이스가 필요로 하는 테라폼 버전을 사용하도록 하는 테크닉이 필요.

이러한 문제를 필요한 도구로 다음 두 가지 도구가 있다.

* [**tfswitch(new)**](https://tfswitch.warrensbox.com/)
* [tfenv(old)](https://github.com/tfutils/tfenv)

위 도구들이 하는 역할은 **.terraform-version** 파일을 찾고 해당 파일의 테라폼 버전을 읽어서 해당 버전의 테라폼을 설치한다. 
