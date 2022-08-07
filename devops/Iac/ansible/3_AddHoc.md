# Ansible - AddHoc

## 앤서블을 사용하는 방법

* Ansible playbook
* Ansible-pull
* ad-hoc command : 명령어라서 간단하지만 재사용하기 어렵다. 그래서 테스트 목적으로 사용된다.

## Addhoc 기본

### ping

대상 호스트에 연결 후 파이썬 사용 가능 여부 확인

```shell
ansible -i amazon.inv -m ping all -u ec2-user
```

* `-i amazon.inv` : 인벤토리로 `amazon.inv`를 지정
* `-m ping` : ping module 선택
* `all` : 모든 그룹 선택
* `-u ec2-user` : ec2-user로 ssh 접속
* `--private-key=~/.ssh/dev.pem` : ssh-add로 .pem 키를 추가하지 않다면 `--private-key`로 지정

### command

해당 운영체제에서 직접 명령을 수행한다. `uptime` 명령어로 호스트들이 얼마동안 실행됬는지 확인

```shell
ansible -i vars.inv -m command -a "uptime" ubuntu
```

* `-a "uptime"` : update 명령 실행
* `ubuntu` : ubuntu 그룹 선택

### setup

상세(Facts) 수집 모듈. 리모스 호스트의 정보 수집 모듈

```shell
ansible localhost -m setup
```

* `localhost` : Ansible에서 모든 인벤토리는 기본적으로 all과 localhost가 정의되어 있다. localhost는 실제 작업하고 있는 로컬 데스크탑을 의미

### apt

apt 명령어를 통해서 패키지 관리

```shell
ansible -i vars.inv -m apt -a "name=git state=latest update_cache=yes" ubuntu --become
```

* `-a "name=git state=latest update_cache=yes"`
  * `name=git` : 패키지 이름
  * `state=latest` : 패키지를 최신 상태로(설치)
  * `state=absent` : 패키지를 삭제
  * `update_cache=yes` : 캐시 업데이트
* `ubuntu` : ubuntu 그룹 선택
* `--become` : root 권한으로 승격

## Control Node VS Managed Node

Contrl node : 앤서블 명령을 수행하는 공간

* 여기서는 로컬 데스크탑
* Ansible이 설치되어 있어야 함

Managed node

* aws ec2 amazon os & ubuntu os
* 파이썬이 설치되어 있어야 함
