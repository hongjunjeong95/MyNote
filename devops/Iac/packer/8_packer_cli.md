# Packer - CLI

**Ref**

* [CLI](https://www.packer.io/docs/commands)

## Commands

* `packer fmt .` : 패커 컨벤션에 맞게 코드를 고쳐줌
* `packer inspect .` : 해당 템플릿이 어떤 구조를 가지고 있는지 확인
* `packer validate .` : 각각의 속성의 타입이 타당한지 확인 등, 템플릿이 valid한지 확인
* `packer version` : 패커 버전 출력
* `packer fix` : 패커가 버전 업데이트가 되면서 패커 문법이 조금씩 개선이 되는데 그것을 자동으로 고쳐줌
* `packer hcl2_upgrade` : JSON 템플릿으로 사용하던 패커를 HCL 코드로 바꿔줌
* `packer console` : 패커에서 쓰이고 있는 HCL 문법을 간단하게 expression으로 테스트해볼 수 있다.
* `packer build -debug .` : 패커로 디버깅 한다.
