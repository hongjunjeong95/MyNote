# Packer - Builder

**Ref**

* [builders](https://www.packer.io/docs/builders)

어떤 종류의 머신 이미지(aws ec2 ami, docker ami or GCP image)를 만들 것인지 결정

## Example

### versions.pkr.hcl

```json
packer {
  required_version = "~> 1.7"

  required_plugins {
    amazon = {
      version = "~> 1.0"
      source  = "github.com/hashicorp/amazon"
    }
  }
}
```

테라폼처럼 버전을 명시

### sources.pkr.hcl

```json
source "null" "one" {
  communicator = "none"
}

source "null" "two" {
  communicator = "none"
}
```

source는 builder를 의미한다. 여기서 `null` builder를 이용하고 있다.

* communicator : provisioning을 할 때 ssh 통신을 정의

### main.pkr.hcl

```json
/**
 * Without Name
 */
build {
  sources = [
    "source.null.one",
    "source.null.two",
  ]
}

/**
 * With Name
 */
build {
  name    = "fastcampus-packer"

  sources = [
    "source.null.one",
    "source.null.two",
  ]
}

/**
 * Fill-in
 */
# extend는 가능하지만 overwrite는 불가능하다.
build {
  name    = "fastcampus-packer-fill-in"

  source "null.one" {
    name = "terraform"
  }

  source "null.two" {
    name = "vault"
  }
}
```

<img src="https://user-images.githubusercontent.com/92770273/146325396-0fb7c1ba-ba74-41c1-b67b-ac8b5a0dce40.png" alt="image" style="zoom:50%;" />

**With name**에서처럼 name을 추가하면 **fastcampus-packer.*** 으로 output이 출력된다.

* `build` block에서 속성을 extend할 수 있지만 overwrite를 할 수는 없다.

## 선택적 빌드(only & except)

### only

```shell
packer build -only="null.two,fastcampus-packer.null.two" .
packer build -only="*.one" .
```

위 프로세스만 빌드를 한다.

### except

```shell
packer build -except="null.two,fastcampus-packer.null.two" .
packer build -except="*.one" .
```

해당 프로세스를 제외하고 빌드 한다.
