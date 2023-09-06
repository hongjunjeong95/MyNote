# 파드의 배치 전략 - Node Selector

Node Selector를 이용하여 파드를 어떤 노드에 배치할 수 있는지 정할 수 있다.

## minikube로 멀티노드 클러스터 구성

minikube는 멀티 노드 클러스터를 지원한다.

기존 minikube 클러스터 제거 : `minikube delete`

- 싱글 노드로 생성된 minikube 클러스터는 기본적으로 CNI(Container Network Interface)를 설치하지 않아, 재구성하는게 간단하다.

노드 3개로 구성된 minikube 클러스터 구성(docker driver 이용)

```sh
minikube start --nodes 3 --driver=docker
```



## nodeName을 이용한 배치

<strong style="color:#eb4444">매니페스트의 재활용성이 떨어지며, 노드와 강결합되기 때문에 추천되지 않는 방법</strong>

- 노드의 이름 기반으로 파드가 배치될 노드를 결정

```yaml
spec:
  containers:
  - name: nginx
    image: nginxdemos/hello:plain-text
    ports:
    - name: http
      containerPort: 80
      protocol: TCP
  nodeName: minikube
```



## nodeSelector를 이용한 배치

- 노드도 쿠버네티스 API 오브젝트로 관리되기 때문에 Labels을 가지고 있음
- 노드에 설정된 Label을 기반으로 하여 Label Selector 기반 파드 배치 가능

```yaml
spec:
  containers:
  - name: nginx
    image: nginxdemos/hello:plain-text
    ports:
    - name: http
      containerPort: 80
      protocol: TCP
  nodeName:
  	team: red
```



## 노드 Label 관리

- 노드를 새로 구성할 때 kubelet 옵션을 통해서 기본 Labels 설정도 가능
- kubectl을 통해 노드의 label 관리 가능

### 명령어

- minikube-m02 노드에 team=key Label 추가

  ```sh
  kubectl label node minikube-m02 team=red
  ```

- minikube-m02 노드에 team Label 제거

  ```sh
  kubectl label node minikube-m02 team-
  ```

  
