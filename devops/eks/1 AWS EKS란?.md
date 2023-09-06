# AWS EKS 소개

## AWS EKS란?

### AWS의 관리형 쿠버네티스 클러스터

- 제어 영역(Control Plane)을 직접 프로비저닝하거나 관리하지 않아도 되는 편의성
- 워크로드가 실행되는 노드 구성에 대한 자유도
- 여러 가용 영역(AZ) 기반의 고가용성 제공
- 편리한 클러스터 버전 업그레이드
- AWS 서비스들과의 통합

<img width="1081" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/7f7ba6ce-da93-41f2-9ff2-05267b467e9a">



## AWS EKS 아키텍쳐

### 쿠버네티스 클러스터 구성

<img width="874" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/dc9d5ab2-63b4-431c-aa58-38da26fdf873">

1. etcd : DB 역할을 한다.
2. kube-api-server : 클라이언트와 통신을 한다.

### EKS 구성

<img width="734" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/41f48b04-8e0e-485e-be48-a2013ad4b37b" style="zoom:67%;" >

- Customer VPC가 AWS 사용자를 의미한다.
- ENI라는 네트워크 인페이스, 즉 private link라는 기술을 AWS EKS VPC에 존재하는 Control Plane이 Customer VPC와 통신을 할 수 있게 한다.



## 로드 밸런싱

### 서비스

- LoadBalancer 타입 서비스
  - 기본적으로 AWS ELB의 CLB(Classic Load Balancer)로 구성
  - Annotation 설정을 통해 AWS ELB의 NLB(Network Load Balancer)로 구성 가능

### 인그레스

- aws-load-balancer-controller 추가 애드온 설치 필요
- AWS ELB의 ALB(Application Load Balancer) 구성

<img width="857" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/151a8002-e238-4dc2-9475-50cb5bbdccc1">



## 파드 네트워킹

### AWS VPC CNI

- AWS에서 기본 제공하는 쿠버네티스 네트워크 애드온
- AWS EC2는 VPC의 ENI를 통해 IP 할당
- ENI의 Secondary IP Addresses를 통해 노드 내 파드가 동일한 VPC IP 대역에서 파드 IP 할당
  - 때문에 ENI의 IP 주소 개수 제한만큼 노드의 파드 개수가 제한됨
- <strong style="color:#eb4444">CNI는 Container Network Interface의 약자로, 리눅스 컨테이너의 네트워크 인터페이스 설정을 위한 스펙 AWS VPC CNI는 AWS VPC를 기반으로 만들어진 CNI 구현체</strong>

<img width="984" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/bff58ba7-9e0c-4476-9854-9bd60e4581b8">



## 저장소

### EBS Built-in StorageClass

- gp2 타입 EBS에 대한 StorageClass 내장되어 바로 사용 가능

### EBS & EFS CSI Driver

- CSI 드라이버 추가 구성을 통해 EBS / EFS 사용 및 최신 기능 활용 가능



## 로그 및 메트릭

### 제어 영역

- 클러스터 생성 시 로깅 옵션을 통해 API Server, Audit, Authenticator, Scheduler, Controller Manager에 대한 로그를 CloudWatch Logs로 수집 가능

### 노드(Node)

- 기본 기능을 제공하지는 않지만, Fluentd나 Fluent Bit을 통해 CludWatch Logs로 파드 로그 전송 가이드 제공



## EKS를 사용해야 하는 이유

### 클러스터 운영보다는 실제 비즈니스 서비스 운영에 집중

- 쿠버네티스 제어영역을 이해하고 직접 관리하는 것이 쉬운 일은 아님
- 비즈니스 성격에 따라 다르겠지만, 클러스터 운영과 비즈니스 서비스 운영 중 무엇이 중요한지 고민 필요

### 인프라 비용을 넘어 관리 비용도 고려

- 직접 쿠버네티스 제어영역을 프로비저닝하여 관리하는게 인프라 비용은 더 쌀 수 있음
- 하지만, 시스템을 관리하는 인력 및 시간에 대한 비용도 고려 필요
