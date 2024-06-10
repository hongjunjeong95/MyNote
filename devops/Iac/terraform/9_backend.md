# Terraform - Backend

**Ref**

* backend : https://www.terraform.io/docs/language/settings/backends/index.html

## 용어

* 워크스페이스 : 상태를 관리하는 단위
* State : 변경사항
* .terraform.tfstate : 상태를 관리하는 파일
* 벡엔드(State Storage)
  * Local Backend
  * Remote Backend(Terraform Cloud)
  * AWS S3 Backend(with / without DynamoDB)
    * with DynamoDB : lock 기능 제공
    * without DynamoDB : lock 기능 제공 안함
* Locking : 테라폼 상세 파일을 원격으로 협업할 때, 동시성 이슈가 생길 수 있다. 이것을 막아주는 것이 locking이다. 한 명이 작업할 때 lock을 걸어서 다른 사람이 apply를 하지 못 하도록 한다.

## Local State

`tf init` 명령어로 로컬에 생성되는 `.terraform.tfstate `

## Remote State

AWS S3나 Terraform Cloud를 이용해서 원격으로 .terraform.tfstate를 관리한다.

## AWS S3

```json
terraform {
  backend "s3" {
    bucket = "<bucket name>"
    key    = "<path>/terraform.tfstate"
    region = "ap-northeast-2"
  }
}
```

`tf init` 명령어로 .terraform.tfstate 파일을 s3 bucket에 저장하게 된다.

## Terraform Cloud

```json
terraform {
  backend "remote" {
    hostname     = "app.terraform.io"
    organization = "<org name>"

    workspaces {
      name = "<workspace name>" 
    }
  }
}
```

### Terraform token 등록하기

<img src="https://user-images.githubusercontent.com/92770273/146123221-7690585c-6322-4662-ac7e-56d4aa33ab72.png" alt="image" style="zoom:50%;" />

**User settings** 클릭

<img src="https://user-images.githubusercontent.com/92770273/146123364-2813b611-f879-42fb-a382-1ae036e58aac.png" alt="image" style="zoom:40%;" />

1. 좌측 **Tokens** 클릭
2. **Create an API token**

<img src="https://user-images.githubusercontent.com/92770273/146123476-be2b4c21-bb5a-44c0-8cf0-d690e52ed53e.png" alt="image" style="zoom:50%;" />

1. Description 입력
2. **Create an API token**

<img src="https://user-images.githubusercontent.com/92770273/146123585-dd3be2ba-a510-4c7f-862f-89baaa66c0f2.png" alt="image" style="zoom:50%;" />

창을 닫으면 토큰 값을 다시 displayed 할 수 없으니, 반드시 **복사**한다.

Terraform cloud에 state를 저장하기 위해서 `~/.terraformrc` 파일에 토큰 값을 저장해야 한다.

```
plugin_cache_dir   = "$HOME/.terraform.d/plugin-cache"

credentials "app.terraform.io" {
  token = "<token>"
}
```

`tf init` 명령어로 terraform cloud에 workspace 생성

### AWS Credentials 등록하기(Remtoe exec mode)

**Execution Mode**를 Remote로 했을 경우 AWS CLI와 연동하기 위해서 variables에 **AWS_ACCESS_KEY_ID**와 **AWS_SECRET_ACCESS_KEY**를 등록해줘야 한다.