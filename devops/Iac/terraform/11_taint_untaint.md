# Terraform - taint & untaint

**Ref**

* taint : https://www.terraform.io/docs/cli/commands/taint.html
* untaint : https://www.terraform.io/docs/cli/commands/untaint.html

## Commands

* `tf taint <resource name>` : 리소스를 비정상적인 상태라고 표시
  * 인프라 리소스에서 변경 사항은 없는데, 무언가 이것 때문에 동작은 안 하고 있는 것 같아서 해당 리소스를 교체하거나 디버깅을 하고 싶을 때 사용한다.
* `tf untaint <resource name>` : taint 해제
* `tf apply -replace=<resource name>` : 특정 리소스를 강제로 교체

## taint 동작 원리

하나의 리소스에 taint를 걸고 `tf apply`를 했을 때 taint가 된 리소스에 의존성을 가진 다른 리소스들도 replaced가 된다.



