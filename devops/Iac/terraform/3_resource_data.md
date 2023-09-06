# Terraform - resource and data

**Ref**

* https://www.terraform.io/docs/language/resources/index.html

## Example

```json
provider "aws" {
  region = "ap-northeast-2"
}

data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical
}

resource "aws_instance" "ubuntu" {
  ami           = data.aws_ami.ubuntu.image_id
  instance_type = "t2.micro"

  tags = {
    Name = "fastcampus-ubuntu"
  }
}
```

* Resource : 데이터를 쓰는 용도
* Data source : 데이터를 읽는 용도

## Argument Refenrece

<img src="https://user-images.githubusercontent.com/92770273/145943127-d7c35b07-c8d9-47f2-965d-6e709ffe4755.png" alt="image" style="zoom:50%;" />

위 인자로 ec2 image를 필터링할 수 있다.

## Attributes Reference

<img src="https://user-images.githubusercontent.com/92770273/145943246-ddac5843-da2f-4947-b580-41d8997f8018.png" alt="image" style="zoom:50%;" />

`output`을 통해서 `arn`와 `architecture` 값을 얻을 수 있다.

### 주의 사항

> resource를 변경했을 때 changed가 아닌 destroy후 created가 될 때가 있다. 예를 들어 vpc의 cidr의 값을 변경했을 때 cidr은 aws vpc에서 변경될 수 있는 값이 아니기 때문에 해당 vpc가 삭제되고 생성된다.
