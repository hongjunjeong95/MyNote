# Terraformer

테라폼 명령어가 아니다. 테라폼과 반대로 이미 생성된 리소스를 테라폼 코드로 만드는 패키지다.

## Installation

1. `mkdir -p ~/.terraform.d/plugins/darwin_arm64`
2. `curl -LO https://releases.hashicorp.com/terraform-provider-aws/5.30.0/terraform-provider-aws_5.30.0_darwin_arm64.zip`
3. `unzip terraform-provider-aws_5.30.0_darwin_arm64.zip`
4. `mv terraform-provider-aws_v5.30.0_x5 ~/.terraform.d/plugins/darwin_arm64/terraform-provider-aws_v5.30.0`
5. `brew install terraformer`

## Docs

https://github.com/GoogleCloudPlatform/terraformer

## Usage

```sh
terraformer import aws --resources=<resource_name>,<resource_name> --path-pattern="{output}/" --connect=true --regions=ap-northeast-2

# example
terraformer import aws --resources=vpc_peering --path-pattern="{output}/" --connect=true --regions=ap-northeast-2
```

### Migration

```yaml
terraform {
    cloud {
        organization = "cookiedeal-tech"

        workspaces {
        tags = ["apne2-cookiedeal-s3"]
        }
    }
}

tf init
- Remove lock file and .terraform folder

# If you want to replace provider hashicorp/aws
tf state replace-provider -- -/aws hashicorp/aws
```
