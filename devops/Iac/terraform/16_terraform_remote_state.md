# Terraform - Terraform Remote State

**Ref**

* [Terraform Remote State](https://www.terraform.io/language/state/remote-state-data)

## Module vs Terraform Remote State

**Module**은 다른 사람이 미리 만들어 놓은 것을 가져와서 리소스를 생성한다면 **Terraform Remote State**는 내가 만들어 놓은 리소스를 가져와서 의존성을 가진다. 모듈은 클래스 같다고 생각하면 된다. 모듈을 가져와서 리소스라는 인스턴스를 만든다면 **Terraform Remote State**는 미리 만들어진 리소스(인스턴스)에 의존성을 가지는 새로운 리소스를 생성한다.

## Example Usage (`local` Backend)

```hcl
data "terraform_remote_state" "vpc" {
	backend = "local"
  config = {
  	path = "..."  
  }
}

# Terraform >= 0.12
resource "aws_instance" "foo" {
	# ...
  subnet_id = data.terraform_remote_state.vpc.outputs.subnet_id
 }
```

## Example Usage (`remote` Backend)

```hcl
data "terraform_remote_state" "vpc" {
	backend = "remote"
  config = {
  	organization = "hashicorp"
    workspaces = {
    	name = "vpc-prod"    
    }  
  }
}

# Terraform >= 0.12
resource "aws_instance" "foo" {
	# ...
  subnet_id = data.terraform_remote_state.vpc.outputs.subnet_id
}
```

## Example Usage (`S3` Backend)

```hcl
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = "hashicorp"
    key    = "vpc-prod"
    region = "ap-northeast-2"
  }
}

# Terraform >= 0.12
resource "aws_instance" "foo" {
	# ...
  subnet_id = data.terraform_remote_state.vpc.outputs.subnet_id
}
```

