# Packer - Post Processor

**Ref**

* [Post-Processor](https://www.packer.io/docs/post-processors)

post-processors는 post-processor를 sequential하게 가진다.

## 패커 동작 순서

패키 => 빌드 => 산출물(Artifact) - image나 manifest file 등 => post-processor => 산출물

## 많이 사용되는 post-processor

* [Checksum(무결성 검증)](https://www.packer.io/docs/post-processors/checksum)
* [Compress(압축)](https://www.packer.io/docs/post-processors/compress)
* [Docker](https://www.packer.io/docs/post-processors/docker/docker-import)
* [Manifest](https://www.packer.io/docs/post-processors/manifest)
* [Local Shell](https://www.packer.io/docs/post-processors/shell-local) : 수동으로 artifact를 후처리하고 싶을 때 사용

## Example

```json
build {
  name    = "fastcampus-packer"

  source "amazon-ebs.ubuntu" {
    name     = "nginx"
    ami_name = "fastcampus-packer"
  }

  post-processor "manifest" {}

  post-processors {
    post-processor "shell-local" {
      inline = ["echo Hello World! > artifact.txt"]
    }
    post-processor "artifice" {
      files = ["artifact.txt"]
    }
    post-processor "compress" {}
  }

  post-processors {
    post-processor "shell-local" {
      inline = ["echo Finished!"]
    }
  }
}
```

**post-processor**와 **post-processors**의 첫 번째 post-processor는 빌드 산출물을 direct로 입력 받을 수 있다. post-processors에서 두 번째 post-processor부터는 빌드 산출물이 아닌 이전 단계의 값을 입력으로 받는다. 하지만 위에서 "shell-local" 같은 경우 명령어를 실행하기 때문에 산출물을 만들지는 않는다. 하지만 **artifice** post-process를 사용하면 "shell-local"에서 생성한 artifact.txt를 compress post-processor에게 산출물로서 입력을 줄 수 있다.
