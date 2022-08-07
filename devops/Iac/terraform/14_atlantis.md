# Terraform - Atlantis

**Ref**

* [Atlantis](https://www.runatlantis.io/guide/#overview-%E2%80%93-what-is-atlantis)
* [Atlantis Github](https://github.com/runatlantis/atlantis)

Github 같은 VSC에서 코드 리뷰를 진행할 때 테라폼을 인터그레이션 해가지고 PR 상에서 테라폼 plan 결과를 확인하고 사람들이 코드 리뷰를 approve 함에 따라서 테라폼 apply를 자동으로 할 수 있게 도와준다.

PR 상에서 comment로 `atlantis plan`을 남기면 atlantis github bot이 인식을 해서 해당 변경 사항에 대해서 테라폼 plan 결과를 출력한다. `atlantis apply` 명령을 실행하면 complience를 체크하고 atlantis policy로 설정된 Approval 개수를 만족하면 apply를 수행하고 정상적으로 수행된 결과를 log file을 comment로 확인 가능
