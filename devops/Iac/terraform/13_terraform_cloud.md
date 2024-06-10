# Terraform - Terraform cloud

## Workspaces

### Settings => Execution Mode

테라폼 명령어를 어디서 실행할 것인지 선택

#### Local

작업자 pc에서 명령어를 실행하고 terraform cloud는 상태 저장만 수행한다.

#### Remote

Terraform cloud 인프라에 위치한 테라폼 명령어를 실행시키는 terraform runner가 있다.

테라폼 명령어를 remote로 실행하는 것에서 주의사항이 있다. 테라폼 코드가 인트라넷에 속한 작업자 pc에 있다면 terraform runner가 인트라넷에 위치한 리소스에 접근할 수 있도록 채널을 열어주는 작업을 해줘야 하기 때문이다.

* Apply Method : Terraform plan이 성공했을 때 자동으로 apply를 할지 말지 선택
* Terraform Version 선택
* Terraform Working Directory 설정 : 현재 작업 중인 Git 레포지토리 전체를 테라폼 클라우드 runner에 업로드하게 된다. 그래서 레포지토리 상에서 어떤 working directory에서 작업 중이냐고 설정해줘야 한다.

**Run** 탭에서 execution log를 확인할 수 있다.

### Variables

Setting => Execution Mode => Remote를 선택했을 때 생기는 탭이다. `terraform.tfvars`의 변수들을 UI를 통해서 등록.

AWS Provider를 사용하기 때문에 AWS credentials를 환경변수로 설정해줘야 remote run을 할 수 있다.

![image](https://user-images.githubusercontent.com/92770273/146134898-e60f4490-dd86-4e7a-be39-162f6632c31f.png)

* 환경 변수는 **Sensitive**를 체크
* Value 값이 기본적으로 string으로 인식되기 때문에 list로 인식하게 하고 싶다면 **HCL**을 체크

### Overview => Run triggers

Terraform Cloud에서 관리하는 워크스페이스가 많을 수가 있다. 이 때 a 워크 스페이스가 b, c 워크 스페이스를 참조한다고 했을 때 b나 c 워크 스페이스가 변경되면 a 워크 스페이스도 변경이 되야 한다. 그래서 a의 **Run trigger**로 b나 c 워크 스페이스를 등록해주면 b나 c가 apply를 하게 될 경우에 a도 자동으로 apply를 한다.

## Registry

Registry에 모듈을 등록할 수 있다. Terraform registry와는 다르다. Terraform registry는 public하게 접근할 수 있지만 terraform cloud registry는 해당 organization이 접근하는 private한 registry다. Terraform Cloud가 이를 organization에게 무료로 제공하고 있다.

## Settings

Organization에 관한 설정들을 다룰 수 있다.

