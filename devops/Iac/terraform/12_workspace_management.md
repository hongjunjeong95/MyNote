# Terraform - workspace management

**Ref**

* workspace : https://www.terraform.io/docs/language/state/workspaces.html

SW Engineering으로 코드와 데이터는 항상 분리해야 한다. 그래서 테라폼 코드는 코드로 남고 데이터는 **foo.tfvars**로 관리를 한다.

워크 스페이스는 깃의 브랜치와 유사하지만 똑같지는 않다. 왜냐하면 브랜치를 바꿀 때마다 코드가 바꾸지만 테라폼에서는 워크스페이스를 바꿀 때마다 내가 수정한 코드들이 바뀌는 것은 아니기 때문이다. 대신 워크스페이스를 옮겨 다니고 *.tfvars 파일을 다른 것으로 명시해줄 때 서로 다른 리소스들을 생성할 수 있다.

## Use cases

* dev / staging / prod 로 나눠서 워크 스페이스를 관리
* kr / jp / us 로 나눠서 국가별로 관리

## Commands

* `tf workspace list` : List workspaces
  * default workspace는 기본적으로 생성된 워크 스페이스다.
  * (*) 표시가 있으면 현재 워크 스페이스를 뜻한다.
* `tf workspace show` : 현재 테라폼 프로젝트에서 설정된 워크 스페이스가 무엇인지 보여줌
* `tf workspace new <workspace name>` : Create a new workspace
* `tf workspace select` : select a workspace
* `tf workspace delete <workspace name>` : Delete a workspace

## Usage

dev, staging, prod 워크스페이스 등을 생성하고 각각의 `dev.tfvars`, `staging.tfvars`, `prod.tfvars` 를 생성해서 데이터를 관리한다. 해당 워크 스페이스에서 `tf apply`를 하고 싶다면 `tf apply -var-file=dev.tfvars`를 통해서 원하는 변수 값을 선택한다.

## 주의 사항

테라폼 클라우드를 이용할 때는 다르게 동작하니 공식 문서를 살펴보자
