# AWS EKS 클러스터 구성 - 노드그룹 생성

## 관리형 노드그룹과 사용자 관리 노드

### 관리형 노드그룹(Managed NodeGroup)

- ASG(Auto-scaling Group)와 LaunchTemplate 기반
- EKS가 EC2 인스턴스의 프로비저닝과 라이프사이클 관리
- 안전한 버전 업그레이드 및 노드 종료 등 지원(실행중인 파드의 안전한 재배치)

### 사용자 관리 노드(Self-managed Nodes)

- AWS EKS에서 제공해주는 EC2 AMI 사용
- 설정 방법에 대한 자유도가 높은 대신 직접 관리해야 할 요소가 많음



## 사전 준비

### IAM Role for Node Group

- EKS 클러스터의 Node Group이 사용할 Service Role 필요
- EC2 머신으로 구성되기 때문에 Trusted Identity는 EC2 서비스 선택
- 다음의 정책(Policy) 연결 필요
  - "arn:aws:iam::aws:policy/AmazoneEKSWorkerNodePolicy"
  - "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
  - "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"

### Security Group for Control Plane

- EKS 클러스터 생성시 기본적으로 제공되는 Cluster Security Group을 연결하면 Control Plane - Node 간 연결 문제 해결
- EKS 노드에 SSH 접속 등 추가 규칙이 필요하면 사용자 정의 보안그룹 생성하여 노드그룹에 연결
  - <strong style="color:#eb4444">관리형 노드그룹은 SSH 옵션을 통해 SSH 접근 통제를 위한 보안그룹 자동 관리</strong>


<strong style="color:#eb4444">노드 생성은 20~30분 정도 소요</strong>



## 노드 생성 후 해야하는 작업

aws-auth ConfigMap에 EKS 노드의 IAM Role 등록

![image](https://github.com/tedilabs/fastcampus-devops/assets/33750210/ec23df3b-3321-4e9e-91a6-d1ba09a1c70e)

<strong style="color:#eb4444">관리형 노드는 자동으로 awa-auth ConfigMap에 등록</strong>



## 테라폼 모듈 구성

### AWS Launch Template / AWS AutoScaling Group

테라폼 리소스 문서

- [aws_launch_template](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/launch_template)
- [aws_autoscaling_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_group)

[테라폼 레지스트리(모듈)](https://registry.terraform.io/modules/tedilabs/container/aws/latest/submodules/eks-node-group)
[GitHub 레포지토리](https://github.com/tedilabs/terraform-aws-container/tree/v0.13.0/modules/eks-node-group)

리소스 구성

- Launch Template - Auto-scaling Group의 설정 목적
- Auto-scaling Group - EKS AMI 기반으로 EC2 노드 생성 목적

### Kubernetes Config Map

테라폼 리소스 문서

- [kubernetes_config_map](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/config_map)

[테라폼 레지스트리(모듈)](https://registry.terraform.io/modules/tedilabs/container/aws/latest/submodules/eks-aws-auth)
[GitHub 레포지토리](https://github.com/tedilabs/terraform-aws-container/tree/v0.13.0/modules/eks-aws-auth)

리소스 구성

- ConfigMap - kube-system 네임스페이스의 aws-auth ConfigMap 구성. 클러스터 인증 제어 목적

