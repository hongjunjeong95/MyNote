# Terraform - Provisioner and EC2 userdata

**Ref**

* [Declaring Provisioners](https://www.terraform.io/language/resources/provisioners/syntax)
* [Terraform provisioner](https://www.terraform.io/language/resources/provisioners/syntax)

## AWS EC2 Userdata

* cloud-init(Linux 지원) : 부팅시점에 user_data를 가지고 bootstraping을 할 수 있음.
  * Bootstraping 하는 것들
    * OS의 사용자 생성
    * 파일 구성
    * SW 설치

### 실행 시점

* 첫 부팅 시점에만 실행

### Example

```json
###################################################
# Userdata
###################################################

# user data 같은 경우는 ec2에서 제공해주는 기능이기 때문에
# terraforma에 script 실행에 대한 로그가 남지 않는다.
resource "aws_instance" "userdata" {
  ami           = data.aws_ami.ubuntu.image_id
  instance_type = "t2.micro"
  key_name      = "dev"

# user data script는 ec2 instance가 첫 부팅될 때만 사용할 수 있다.
# 만약 파일을 수정하고 `tf apply`를 하면 terraform이 instance를 삭제하고 다시 설치하냐고 물어본다.
  user_data = <<EOT
#!/bin/bash
sudo apt-get update
sudo apt-get install -y nginx
EOT

  vpc_security_group_ids = [
    module.security_group.id,
  ]

  tags = {
    Name = "fastcampus-userdata"
  }
}
```

* user data 같은 경우는 ec2에서 제공해주는 기능이기 때문에
* terraforma에 script 실행에 대한 로그가 남지 않는다.
* **주의사항**
  * user data script는 ec2 instance가 첫 부팅될 때만 사용할 수 있다.
  * 만약 파일을 수정하고 `tf apply`를 하면 terraform이 instance를 삭제하고 다시 설치하냐고 물어본다.

## Terraform Provisioner

### 종류

* file : local에서 remote로 파일 복사
* local exec : local 머신에서 명령어 수행
* remote exec : remote 머신에서 명령어 수행
  * ssh(Unix/Linux) 지원
  * win_rm(window)
* salt(deprecated)
* chef(depreacted)

### 실행 시점

* 첫 리소스 시점
* 여러 옵션을 통해서 삭제 시점 또는 매번 수행

### Example

```json
# provisioner 같은 경우 terraform에서 제공해주는 기능이기 때문에
# ec2에 connection을 맺는 것부터 script가 실행되는 로그가 전부 남는다.
resource "aws_instance" "provisioner" {
  ami           = data.aws_ami.ubuntu.image_id
  instance_type = "t2.micro"
  key_name      = "dev"

  vpc_security_group_ids = [
    module.security_group.id,
  ]

  tags = {
    Name = "fastcampus-provisioner"
  }

	provisioner "local-exec" {
    command = "echo Hello World"
  }

  provisioner "file" {
    source      = "${path.module}/files/index.html"
    destination = "/tmp/index.html"

    connection {
      type = "ssh"
      user = "ubuntu"
      host = self.public_ip
    }
  }

  # provisioner는 기본적으로 creation-time provisioner다.
  # 즉 instance가 생성될 때마다 실행된다. sciprt를 추가해도 userdata와 달리 아무런 변화가 없다.
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
    ]
    # inline말고도 script, scripts를 사용할 수 있는데
    # script는 로컬에 있는 script 파일을 올린다.

    connection {
      type = "ssh"
      user = "ubuntu"
      host = self.public_ip
      # self는 provisioner에서만 사용할 수 있는 변수다.
    }
    # ssh-agent를 사용하던가 password나 private_key를 사용해야 한다.
    # 현업에서는 ssh-agent를 사용해야 코드 상에 비밀 데이터가 남지 않아서 보안상 좋다.
  }
}
```

* provisioner 같은 경우 terraform에서 제공해주는 기능이기 때문에 ec2에 connection을 맺는 것부터 script가 실행되는 로그가 전부 남는다.

* provisioner는 기본적으로 creation-time provisioner다. 즉 instance가 생성될 때마다 실행된다. sciprt를 추가해도 userdata와 달리 아무런 변화가 없다.

  <img src="https://user-images.githubusercontent.com/92770273/146309501-44c1e7d3-34fb-4bff-8747-7e8d8b0773e8.png" alt="image" style="zoom:40%;" />

#### local-exec

command로 명령어 작성한다. local pc에 명령어를 내린다.

### file

source(local pc)에 있는 파일을 destination(ec2 instaince)에 복사한다.

### remote-exec

* Script를 작성할 때 inline말고도 script, scripts를 사용할 수 있는데, script는 로컬에 있는 script 파일을 올린다.
* self는 provisioner에서만 사용할 수 있는 변수다.
  * 여기서 self는 `aws_instance.provisioner`를 뜻한다.
* connection을 맺을 때는 ssh-agent를 사용하던가 password나 private_key를 사용해야 한다. 현업에서는 ssh-agent를 사용해야 코드 상에 비밀 데이터가 남지 않아서 보안상 좋다.

### Null Resource

```json
###################################################
# Provisioner - in null-resources
###################################################

resource "aws_instance" "provisioner" {
  ami           = data.aws_ami.ubuntu.image_id
  instance_type = "t2.micro"
  key_name      = "dev"

  vpc_security_group_ids = [
    module.security_group.id,
  ]

  tags = {
    Name = "fastcampus-provisioner"
  }
}

resource "null_resource" "provisioner" {
  triggers = {
    insteance_id = aws_instance.provisioner.id
    script       = filemd5("${path.module}/files/install-nginx.sh")
    index_file   = filemd5("${path.module}/files/index.html")
  }

  provisioner "local-exec" {
    command = "echo Hello World"
  }

  provisioner "file" {
    source      = "${path.module}/files/index.html"
    destination = "/tmp/index.html"

    connection {
      type = "ssh"
      user = "ubuntu"
      host = aws_instance.provisioner.public_ip
    }
  }

  provisioner "remote-exec" {
    script = "${path.module}/files/install-nginx.sh"

    connection {
      type = "ssh"
      user = "ubuntu"
      host = aws_instance.provisioner.public_ip
    }
  }

  provisioner "remote-exec" {
    inline = [
      "sudo cp /tmp/index.html /var/www/html/index.html"
    ]

    connection {
      type = "ssh"
      user = "ubuntu"
      host = aws_instance.provisioner.public_ip
    }
  }
}
```

```json
resource "null_resource" "provisioner" {
  triggers = {
    insteance_id = aws_instance.provisioner.id
    script       = filemd5("${path.module}/files/install-nginx.sh")
    index_file   = filemd5("${path.module}/files/index.html")
  }
}
```

triggers 안에 있는 값이 변경되면 새롭게 re-provisioning을 한다.
