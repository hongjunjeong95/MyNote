# Packer - Provisioner

**Ref**

* [provisioner](https://www.packer.io/docs/provisioners)

## File

로컬 => 리모트로 파일 업로드

## Shell

머신에서 명령어 수행

## Shell-local

local pc에서 명령어 수행

## Breakpoint

디버깅 목적으로 사용한다. 사용자가 ok를 누르기 전까지 해당 포인트에서 멈추다가 ok를 누르면 진행된다.

## Example

```json
build {
  name = "fastcampus-packer"

  source "amazon-ebs.ubuntu" {
    name     = "nginx"
    ami_name = "fastcampus-packer-nginx"
  }

  # provisioner는 정의한 순서가 중요!!

  provisioner "shell" {
    inline = [
      "sudo apt-get update",
      "whoami",
    ]
  }

  provisioner "file" {
    source      = "${path.root}/files/index.html"
    destination = "/tmp/index.html"
  }

  provisioner "shell" {
    inline = [
      # source.name = nginx / source.type = amazon-ebs
      "echo ${source.name} and ${source.type}",
      # "whoami",
      "sudo apt-get install -y nginx",
      "sudo cp /tmp/index.html /var/www/html/index.html"
    ]
  }

  provisioner "breakpoint" {
    disable = false
    note    = "디버깅용"
  }
}
```

