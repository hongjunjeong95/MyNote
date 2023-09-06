# AWS EKS 클러스터 구성 - 클러스터 생성

## 사전 준비

### IAM Role for Control Plane

- EKS 클러스터의 Control Plane이 사용할 Service Role 필요
- 다음의 정책(Policy) 연결 필요
  - "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy" <strong style="color:#eb4444">필수</strong>
  - "arn:aws:iam::aws:policy/AmazonEKSVPCResourceController"

### Security Group for Control Plane

- EKS 클러스터는 생성 시 기본적으로 Cluster Security Group 이라는 기본 보안 그룹 생성
  - Node Gorup, Pod, Fargate 와의 통신에 필요한 내부 보안그룹 규칙 생성
- 사용자 정의 보안 그룹을 클러스터 생성 시점에 연결 가능
  - Cluster Security Group의 제어는 EKS가 하기 때문에 직접 제어할 수 있는 사용자 정의 보안그룹 생성 추천
  - 클러스터 생성 뒤에는 연결 불가

<strong style="color:#eb4444">클러스터 생성은 20~30분 정도 소요</strong>



## 테라폼 모듈 구성

테라폼 리소스 문서 - [aws_eks_cluster](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eks_cluster)

[테라폼 레지스트리(모듈)](https://registry.terraform.io/modules/tedilabs/container/aws/latest/submodules/eks-cluster)
[GitHub 레포지토리](https://github.com/tedilabs/terraform-aws-container/tree/v0.13.0/modules/eks-cluster)

리소스 구성

- EKS Cluster : EKS 클러스터의 제어 영역
- CloudWatch Log Group : EKS 클러스터 제어 영역의 로그 전달 목적
- IAM Role : EKS 클러스터의 제어 영역, 노드, 파게이트가 사용할 수 있는 미리 정의된 서비스 IAM Role
- Security Group : EKS 클러스터의  제어 영역, 노드, 파드에서 사용할 수 있는 미리 정의된 보안 그룹
- IAM OIDC Provider : IRSA(IAM Role for Service Account)를 위한 EKS 클러스터의 OIDC Provider

```sh
# 로컬에서 EKS 클러스터에 연결할 수 있게 한다.
aws eks update-kubeconfig --region ap-northeast-2 --name apne2-fastcampus --alias apne2-fastcampus
```

