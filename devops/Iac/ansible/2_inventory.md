# Ansible - Inventory

## What is?

대상 서버를 관리하는 파일

## 기능

* 그룹 기능 지원 : 한 호스트가 여러 호스트에 속할 수 있음
  * 우분투 그룹을 만들어서 우분트 인스턴스들을 넣을 수 있다.
  * 스테이징(dev, test, prod) 별로 그룹을 묶을 수 있음

## Statice & Dynamic

### Static

* 

### Dynamic

* 오토 스케일링을 사용하게될 경우 ip 정보가 빈번하게 바뀔 수 있음. 그럴 때 활용할 수 있는 것이 dynamic inventory다.

## 인벤토리 기본

```
# amazon.inv
3.35.53.163
3.34.253.165

# ubuntu.inv
ec2-3-34-49-77.ap-northeast-2.compute.amazonaws.com
ec2-13-209-20-108.ap-northeast-2.compute.amazonaws.com

# simple.inv
[amazon]
3.35.53.163
3.34.253.165
[ubuntu]
ec2-3-34-49-77.ap-northeast-2.compute.amazonaws.com
ec2-13-209-20-108.ap-northeast-2.compute.amazonaws.com

# alias.inv
[amazon]
amazon1 ansible_host=3.35.53.163
amazon2 ansible_host=3.34.253.165

[ubuntu]
ubuntu1 ansible_host=ec2-3-34-49-77.ap-northeast-2.compute.amazonaws.com
ubuntu2 ansible_host=ec2-13-209-20-108.ap-northeast-2.compute.amazonaws.com

# vars.inv
[amazon]
amazon1 ansible_host=3.35.53.163 ansible_user=ec2-user
amazon2 ansible_host=3.34.253.165 ansible_user=ec2-user

[ubuntu]
ubuntu1 ansible_host=ec2-3-34-49-77.ap-northeast-2.compute.amazonaws.com ansible_user=ubuntu
ubuntu2 ansible_host=ec2-13-209-20-108.ap-northeast-2.compute.amazonaws.com ansible_user=ubuntu

[linux:children]
amazon
ubuntu
```

* amazon.inv : 인스턴스의 ip를 추가
* ubuntu.inv : 인스턴스의 도메인을 추가
* simple.inv : group 기능을 사용
  * [all]이라는 기본 그룹이 있는데 인벤토리에 정의되어 있는 모든 그룹을 포함
* alias.inv
  * amazon1 등 맨 앞에 있는 것이 별칭이다.
  * 별칭은 유니크해야 한다.
  * 그 뒤에 해당 호스트에 대한 변수를 정의할 수 있음. value로는 ip나 domain을 입력하면 됨
* vars.inv
  * ansible_user로 해당 인스턴스의 기본 사용자를 등록해줘야 한다. 만약 등록하지 않고 바로 로컬에서 인스턴스에 접속을 시도하면 로컬의 계정 정보로 접속으 시도하게 된다. 하지만 로컬의 계정(ex hongjun) 정보는 aws instance에 등록되어 있지 않았기 때문에 `Error`가 발생한다.
  * `[linux:children]` 이 뜻은 linux라는 그룹은 하위(chlidren) 그룹으로 amazon과 ubuntu를 포함한다.
