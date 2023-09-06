# Componets of Clusters on Kubernetes

## 클러스터 구성

Control Plane(Master Node)

- 클러스터의 관리하는 역할 담당
- 상태 관리 및 명령어 처리

Node(Worker Node)

- 어플리케이션 컨테이너 실행

<img width="1337" alt="image" src="https://user-images.githubusercontent.com/33750210/232378894-79fdf6d8-cadd-48ef-9068-9fe2d9a9c4a3.png" style="zoom: 50%;" >

## 제어 영역(Control Plane)

<img src="https://user-images.githubusercontent.com/33750210/232381561-c9d67785-8514-45d5-9248-e25bfd148926.png" alt="image" style="zoom:33%;" />

- API 서버
  - 쿠버네티스 리소스와 클러스터 관리를 위한 API 제공
  - etcd를 데이터 저장소로 사용
- Scheduler
  - 노드의 자원 사용 상태를 관리하며, 새로운 워크로드를 어디에 배포할지 관리
- Controller Manager
  - 컨트롤러 매니저는 쿠버네티스 컨트롤러 매니저와 클라우드 컨트롤러 매니저로 나뉘어져 있다.
  - 여러 컨트롤러 프로세스를 관리
  - 각 컨트롤러는 클러스터로부터 특정 리소스 상태의 변화를 감지하여 클러스터에 반영하는 reconcile 과정을 반복 수행
- etcd
  - 상태관리를 하는 데이터베이스 같은 존재다.
  - 분산 Key - Value 저장소로 클러스터 상태 저장

## 노드(Node)

실제 어플리케이션 컨테이너가 띄워지는 환경

###  kubelet

- 컨테이너 런타임(Container Runtime)과 통신하며 컨테이너 라이프 사이클 관리
- API 서버와 통신하며 노드의 리소스 관리

### CRI(Container Runtime Interface)

- kubelet이 컨테이너 런타임과 통신할 때 사용되는 인터페이스
- 쿠버네티스는 Docker, containerd, cri-o 컨테이너 런타임 지원

### kube-proxy

- 오버레이 네트워크 구성
- 네트워크 프록시 및 내부 로드밸런서 역할 수행
