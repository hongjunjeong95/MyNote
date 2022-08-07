# Packer - Data Source

**Ref**

* [Data Sources](https://www.packer.io/docs/templates/hcl_templates/datasources)

## Data source를 사용하기 전

```json
source "amazon-ebs" "ubuntu" {
  instance_type = "t2.micro"
  region        = "ap-northeast-2"

  source_ami_filter {
    filters = {
      name                = "ubuntu/images/*ubuntu-focal-20.04-amd64-server-*"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["099720109477"]
  }

  ssh_username = "ubuntu"
}
```

## Data source를 사용한 후

```json
data "amazon-ami" "ubuntu" {
  filters = {
    virtualization-type = "hvm"
    name                = "ubuntu/images/*ubuntu-focal-20.04-amd64-server-*"
    root-device-type = "ebs"
  }
  owners = ["099720109477"]
  most_recent = true
}

source "amazon-ebs" "ubuntu" {
  instance_type = "t2.micro"
  region        = "ap-northeast-2"
  source_ami = data.amazon-ami.ubuntu.id

  ssh_username = "ubuntu"
}
```

data source를 사용한 후

```json
source "amazon-ebs" "ubuntu" {
  source_ami = data.amazon-ami.ubuntu.id
}
```

source_ami가 reference 값만 참조하면 되게 되었다. 이렇게 코드와 데이터를 분리하면 유지 관리성이 개선된다. 다른 image를 가져올 때도 data를 이용해서 image id 값만 바꾸면 되기 때문이다.

