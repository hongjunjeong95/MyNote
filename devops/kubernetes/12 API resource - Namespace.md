# API Resource - Namespace

## Namespace이란?

- 쿠버네티스 상의 API 오브젝트들을 논리적으로 구분하여 관리하기 위해 사용한다.
- 분류 뿐만 아니라 해당 논리적인 그룹에 대하여 권한 관리, CPU & Memory 등 리소스 제한 등을 하기 위해 사용한다.

### Namespace

<img width="1165" alt="image" src="https://user-images.githubusercontent.com/33750210/234448835-054af784-c690-4149-81eb-34ff224536ed.png" style="zoom:50%;" >

- 리소스를 논리적으로 나누기 위한 방법 제공 (논리적 그룹)
- 네임스페이스의 단위는 사용자 목적에 맞추어 결정
  - 팀 단위 네임스페이스
  - 환경 단위 네임스페이스
  - 서비스 단위 네임스페이스



## 클러스터 범위 API 리소스와 네임스페이스 범위 API 리소스

네임스페이스에 속할 수 있는 리소스를 네임스페이스 범위 API 리소스라고 함

### 네임스페이스 범위 API 리소스 (Namespace-scoped API Resources)

- Pod, Deployment, Service, Ingress, Secret, ConfigMap, ServiceAccount, Role, RoleBinding 등

  ```sh
  kubectl api-resources --namespaced=true
  ```

### 클러스터 범위 API 리소스 (Cluster-scoped API Resources)

- Node, Namespace, IngressClass, ClusterRole, ClusterRoleBinding 등

  ```sh
  kubectl api-resources --namespaced=false
  ```



## 클러스터 기본 네임스페이스

쿠버네티스 클러스터를 생성하고나면 기본적으로 만들어져 있는 네임스페이스

- default : 네임스페이스를 지정하지 않은 경우에 기본적으로 할당되는 네임스페이스
- kube-system : 쿠버네티스 시스템에 의해 생성되는 API 오브젝트들을 관리하기 위한 네임스페이스
- kube-public : 클러스터 내 모든 사용자로부터 접근 가능하고 읽을 수 있는 오브젝트들을 관리하기 위한 네임스페이스
- kube-node-lease : 쿠버네티스 클러스터 내 노드의 연결 정보를 관리하기 위한 네임스페이스



## 다른 네임스페이스의 서비스 접근하기

서로 다른 서비스가 통신하기 위해서는 서비스명으로는 충분하지 않다. => 네임 스페이스가 다르지만 서비스명이 동일하다면?

### FQDN(Fully Qualified Domain Name)과 Domain Search 옵션

```sh
curl ${service}.${namespace}.svc.cluster.local // FQDN
curl ${service}.${namespace}.svc
curl ${service}.${namespace}
curl ${service} // 동일 네임스페이스 내 서비스 접근시 사용
```

- FQDN : 전체 도메인 이름을 뜻한다. 비유하자면 디렉토리의 절대적인 주소라고 볼 수 있다.
- Domain Search option은 resolv.conf 파일에 설정이 되곤 한다.

 

## ResourceQuota와 LimitRange

네이스페이스 단위의 자원 사용량 괸리할 수 있는 기능 제공

###  ResoruceQuota

- 네임스페이스에서 사용할 수 있는 자원 사용량의 합을 제한
  - 할당할 수 있는 자원(CPU, Memory, Volume 등)의 총합 제한
  - 생성할 수 있는 리소스(Pod, Service, Deployment 등)의 개수 제한

### LimitRange

- 파드 혹은 컨테이너에 대하여 자원 기본 할당량 설정, 혹은 최대 / 최소 할당량 설정
