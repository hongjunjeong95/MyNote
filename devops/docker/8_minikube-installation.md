# minikube installation

**Ref**

* [minikube](https://minikube.sigs.k8s.io/docs/start/)

#### minikube 특징

* 쿠버네티스는 여러 머신을 관리하는 시스템이다. 
* minikube는 복잡한 쿠버네티스 클러스터 구성 작업을 가상환경을 이용하여 쉽게 구성해준다.
* 실제 운영환경에서는 쓰기 어렵지만, 쿠버네티스 학습 목적으로 활용하기 좋다.
* 드라이버(driver)를 선택하여 원하는 가상환경(**docker**, podman, virtualbox, parallels, vmware, hyperkit 등)에서 구성 가능 쿠버네티스 학습환경으로 활용하기 좋음

#### minikube 설치 필요조건

- 2 CPUs or more
- 2GB of free memory
- 20GB of free disk space
- Internet connection
- Container or virtual machine manager, such as: [Docker](https://minikube.sigs.k8s.io/docs/drivers/docker/), [Hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/), [Hyper-V](https://minikube.sigs.k8s.io/docs/drivers/hyperv/), [KVM](https://minikube.sigs.k8s.io/docs/drivers/kvm2/), [Parallels](https://minikube.sigs.k8s.io/docs/drivers/parallels/), [Podman](https://minikube.sigs.k8s.io/docs/drivers/podman/), [VirtualBox](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/), or [VMware Fusion/Workstation](https://minikube.sigs.k8s.io/docs/drivers/vmware/)

## minikube(Ubuntu)

```sh
#!/usr/bin/env bash
## INFO: https://minikube.sigs.k8s.io/docs/start/

set -euf -o pipefail

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### 쿠버네티스 클러스터 구성

```shell
minikube start --driver docker
```

## ~/.kube/config

<img src="https://user-images.githubusercontent.com/33750210/146623411-8cf11a9c-a521-44ba-9c52-b81c38a0921a.png" alt="image" style="zoom:70%;" />

* clusters : 클러스터 정보
* users : 인증 사용자 정보
* contexts : 인증 리스트다. clusters와 users 정보를 조합해서 만든다. 즉 해당 클러스터에 해당 유저 정보로 인증을 한다는 뜻이다.
  * 현재 kubectl이 사용하는 context info가 `current-context`에 기입이 되고 해당 컨테스트 이름은 `name`에서 확인 가능
* `kubectl get nodes` : kubectl이 접속하게 되는 클러스터 정보를 확인 가능
* `kubectl cluster-info` : 
* 

## minikube 기본 사용법

* mnikube status : 쿠버네티스 클러스터 상태 확인
* minikube start : 쿠버네티스 클러스터 시작
* minikube stop : 쿠버네티스 클러스터 중지
* minikube delete : 쿠버네티스 클러스터 삭제
  * 학습 도중 문제가 생기면 걍 삭제하고 다시 시작하는 것도 방법
* minikube pause : 쿠버네티스 클러스터 일시중지(데스크탑의 sleep 같은 기능)
* minikube unpause : 쿠버네티스 클러스터 재개
* minikube addons list : minikube 애드온 목록 확인
* minikube addons enable [addon] : minikube 애드온 활성화
* minikube addons disable [addon] : minikube 애드온 비활성화
* minikube ssh : 쿠버네티스 클러스터 노드에 ssh 접속
* minikube kubectl ... : 쿠버네티스 클러스터 버전과 대응되는 kubectl 사용

### 버전 호환성 확인

```shell
kubectl get node
kubectl version
```

위 두 개의 명령어를 통해서 로컬에 설치된 kubectl의 버전과 kubectl 클러스터 버전이 일치하지 않을 수도 있다. 그러면 호환성 문제가 생기는데, 반드시 이 두 개의 버전을 맞춰줘야 한다.

```shell
minikube kubectl
```

위 명령어로 로컬 버전이 클러스터 버전과 동일한 kubectl 버전을 이용할 수 있도록 제공한다.

## minikube(Mac)

```sh
brew install minikube
```
