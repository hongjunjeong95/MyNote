# Terraform - State Management

**Ref**

* for : https://www.terraform.io/docs/language/expressions/for.html

## Commands

* `tf state list` : 현재 테라폼 워크스페이스에서 관리 중인 리소스 상태들을 확인할 수 있다.
* `tf state show <resource name>` : 리소스의 detail을 코드 형태로 보여줌
* `tf state mv` : 리팩토링을 하는 과정에서 리소스 이름이 변경되었을 때 사용한다. 예를 들어 `aws_iam_group.developer`가 `aws_iam_group.this["developer"]`로 변경됐을 때, 개발자 입장에서는 같은 리소스이지만 terraform은 `aws_iam_group.developer`를 삭제하고 `aws_iam_group.this["developer"]`를 생성한다. 이러한 이슈를 피하기 위해서 `tf state mv`를 통해 마이그레이션을 해준다.
* `tf state rm <resource name>` : 상태 저장소에서 해당 리소스 상태를 제거하는 명령어다. 리소스를 유지하지만 테라폼으로 더 이상 관리하지 않을 경우 사용한다.
* `tf state pull` : remote state store에 있는 상태를 output으로 출력
  * 리모트로 하나의 워크스페이스를 관리하다가 그 워크스페이스가 너무 거대해지면 local state file로 받아서 수동으로 state를 정리해서 여러 워크스페이스로 분리
  * `tf state pull > foo.tfstate`로 상태 파일을 다운
* `tf state push` : local state file을 remote에 덮어씌운다.
  * 굉장히 위험한 명령어이기 때문에 주의!