# API Resource - Service

## 서비스란?

- Deployoment를 통해 파드를 수평확장 하면 트래픽은 어떻게 분산시키지?
- 외부로부터의 요청을 받으려면 어떻게 해야하지?

### Service

<img width="1191" alt="image" src="https://user-images.githubusercontent.com/33750210/233900159-f865a1f6-d2d5-4313-b5a5-e5742f9f04d9.png" style="zoom:50%;" >

- 여러 파드에 대해 클러스터 내에서 사용 가능한 고유 도메인 부여
- 여러 파드에 대한 요청을 분산하는 로드 밸런서 기능 수행
- 파드의 IP는 항상 변할 수 있음에 유의

<strong style="color:#eb4444">일반적으로는 ClusterIP 타입의 Service와 함께 Ingress를 사용하여 외부 트래픽을 처리</strong>



## 서비스의 종류

<img width="2434" alt="image" src="https://user-images.githubusercontent.com/33750210/233900257-3ea52fc1-3020-4e67-878e-6fd2e04c1bf7.png">

### ClusterIP

- 기본값이다.
- 외부 트래픽을 받는 능력은 없다.
- 클러스터 내부에서 서비스로 찔렀을 때, 서비스가 관리하는 파드들에게 로드를 분산시켜주는 역할을 수행한다.

### NodePort

- 외부로부터 트래픽 주입이 가능하다.
- 클러스터 IP를 매핑하는 구조다.
- 클러스터 IP의 모든 기능을 사용할 수 있고 
- 클러스터를 구성하는 각각의 노드에 동일한 포트를 열게 되는데 포트를 통해서 외부 트래픽을 받고 클러스터 IP를 통해서 로드를 분산한다. 

### LoadBalancer

- 서비스와 파드의 구성은 동일하기 때문에 Cluster IP의 기능은 사용할 수 있다.
- 외부에 존재하는 LoadBalancer를 동적으로 관리할 수 있다.
- 보통 클라우드 프로바이더와 함께 사용된다.

### ExternalName

- 다른 세 가지는 트래픽을 받는 용도였다면 ExternalName은 외부로 가는 트래픽을 해석하기 위한 용도다. 즉 도메인을 트랜스레이션 하는 용도다.



## ClusterIP와 서비스 디스커버리

<img width="733" alt="image" src="https://user-images.githubusercontent.com/33750210/233900288-d65d4fd4-7a5d-4cd5-bfc2-7f77968239aa.png" style="zoom:67%;" >

### ClusterIP

- Service API 리소스의 가장 기본적인 타입
- 쿠버네티스 클러스터는 Pod에 부여되는 Pod IP를 위한 CIDR 대역과 Service에 부여되는 Cluster IP CIDR 대역이 독립적으로 존재
- Label Selector를 통해 서비스와 연결할 파드 목록 관리
- Cluster IP로 들어오는 요청에 대하여 파드에 L4 레벨의 로드밸런싱
- Cluster IP 뿐만 아니라 내부 DNS을 통해 서비스 이름을 이용한 통신 가능

### 서비스 네트워크 IP / Port 정보

`spec.clusterIp:spec.ports[*].port`

### 서비스의 Cluster IP CIDER 대역 확인

```sh
kubectl cluster-info dump | grep -m 1 service-cluster-ip-range
```

<strong style="color:#eb4444">ClusterIP 타입의 Service는 쿠버네티스 클러스터 내부 통신 목적으로만 사용 가능</strong>

### Code

<img width="183" alt="image" src="https://user-images.githubusercontent.com/33750210/233906163-f03f9b93-f424-4fa0-8eeb-bc2c2df5c45d.png">

- 여기서 중요한 부분은 `spec.selector`다. 이 옵션은 트래픽을 어떤 pod에게 전달해줄 것인지를 결정한다. 
- ClusterIP service를 생성하면 IP가 자동적으로 생성되고(혹은 spec.clusterIp) port는 clusterIP에 해당하는 port다. 또한, ClusterIP는 클러스터 내에서만 사용할 수 있다. 
  - 대신 `minikube ssh`를 사용해서 curl 요청을 하면 통신을 할 수 있다. 연속해서 클러스터에 요청을 하게 되면 server-name이 계속 바뀌게 되는데 해당 이름 pod들의 이름이다. 즉 클러스터에게 요청을 하면 클러스터는 pod들에게 로드밸런싱을 해준다.



## NodePort로 외부에 노출하기

<img width="717" alt="image" src="https://user-images.githubusercontent.com/33750210/233900326-518b845b-2d3d-44fb-a643-eb1676afff92.png" style="zoom:70%;" >

- 모든 쿠버네티스 노드의 동일 포트를 개방하여 서비스에 접근하는 방식
- NodePort는 ClusterIP 타입 서비스를 한 번 더 감싸서 만들어진 것
  - NodePort 서비스도 ClusterIP 사용 가능
  - NodePort로 들어온 요청은 실제로 ClusterIP로 전달하되어 Pod로 포워딩
- 외부로부터 트래픽 주입이 가능하다.
- 클러스터 IP를 매핑하는 구조다.
- 클러스터 IP의 모든 기능을 사용할 수 있고 
- 클러스터를 구성하는 각각의 노드에 동일한 포트를 열게 되는데 포트를 통해서 외부 트래픽을 받고 클러스터 IP를 통해서 로드를 분산한다. 

<strong style="color:#eb4444">AWS / GCP 등과 같은 클라우드 환경이 아니라면 기본적으로는 해당 기능 이용이 불가.</strong>

<strong style="color:#eb4444">MetalLB와 같은 기술 등을 사용하여 온프레미스 환경에서도 LoadBalancer 타입 사용 가능</strong>

### 서비스 네트워크 IP / Port 정보

- 클러스터 외부에서 접근할 수 있는 NodePort의 IP/Port : `<NodeIP>:spec.ports[*].nodePort`
- 클러스터 내부에서 접근할 수 있는 ClusterIp : `spec.clusterIp:spec.ports[*].port`



## LoadBalancer로 클라우드 프로바이더의 로드밸런서 연동

<img width="536" alt="image" src="https://user-images.githubusercontent.com/33750210/234140669-2abe1684-f5fc-4104-8983-e7a90662ee9c.png">

- 클라우드 프로바이더에서 제공하는 로드밸런서를 동적으로 생성하는 방식
- LoadBalancer 타입 서비스는 NodePort 타입 서비스를 한 번 더 감싸서 만들어진 것
  - LoadBalancer 서비스도 ClusterIP 사용 가능
  - LoadBalancer 서비스를 통해 만들어진 로드밸런서는 NodePort를 타켓 그룹으로 생성
  - NodePort로 들어온 요청은 실제로 ClusterIP로 전달되어 Pod로 포워딩

### 서비스 네트워크 IP / Port 정보

- 로드 밸런서에 접근 : `spec.loadBalancer:spec.ports[*].port`
- 클러스터 외부에서 접근할 수 있는 NodePort의 IP/Port : `<NodeIP>:spec.ports[*].nodePort`
- 클러스터 내부에서 접근할 수 있는 ClusterIp : `spec.clusterIp:spec.ports[*].port`



## ExternalName으로 외부로 요청 전달

<img width="537" alt="image" src="https://user-images.githubusercontent.com/33750210/234140739-41392c1e-f96f-4c38-8930-1f0c65d149b8.png">

- 서비스가 파드를 가리키는 것이 아닌 외부 도메인을 가리키도록 구성 가능
- DNS의 CNAME 레코드와 동일한 역할 수행
- 클러스터의 외부에 존재하는 레거시 시스템을 쿠버네티스로 마이그레이션하는 과정에 활용 가능

<strong style="color:#eb4444">ExternalName 타입의 서비스는 앞서 다룬 세 서비스 타입과 비교해 많이 사용되지는 않는다.</strong>
