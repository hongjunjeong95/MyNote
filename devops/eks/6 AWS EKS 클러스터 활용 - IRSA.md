# AWS EKS 클러스터 활용 - IRSA

## Service Account 소개

파드가 쿠버네티스 클러스터 내 특정 리소스에 대한 권한을 수행하고 싶다면?

* 특정 네임스페이스의 Secret을 읽는 파드
* 클러스터 전체 파드 목록을 확인할 수 있는 파드

### 서비스 어카운트(Service Account)

- 쿠버네티스의 User / Group은 사용자에 대한 클러스터 접근제어를 위한 주체
- 서비스 어카운트는 파드에 대한 클러스터 접근제어를 위한 주체
- 하나의 파드는 하나의 서비스 어카운트에 연결
- 서비스 어카운트를 지정하지 않으면 각 네임스페이스의 default 서비스 어카운트 사용

## IRSA 소개

<img width="765" alt="image" src="https://github.com/tedilabs/fastcampus-devops/assets/33750210/6967d856-ca5b-47d5-9a63-788a2c9cf200" style="zoom:67%;" >

파드가 S3, DynamoDB, SQS와 같은 AWS 서비스에 접근하고 싶다면 어떻게 해야할까?

- EC2는 Instance Profile을 이용
- AccessKey를 만들자니 IAM User를 만들고 기밀 값을 관리해야 함

### IRSA (IAM Role for Service Account)

- 이름처럼 ServiceAccount에 IAM Role을 연결시키는 기술
- ServiceAccount의 metadata.annotations을 통해 IAM Role ARN 명시
- IAM Role 생성 시 EKS 클러스터의 OIDC Provider를 Trusted Entity 등록

## IRSA 사용 준비사항

1. EKS OIDC Provider 정보 확인

   1. EKS 제어영역을 만들고 나면 OIDC Provider 정보 제공

   <img width="923" alt="image" src="https://github.com/tedilabs/fastcampus-devops/assets/33750210/b3f0f5b4-913b-44ff-b596-9c687344c520" style="zoom:67%;" >

2. IAM Identity Provider 등록

   1. IAM 서비스의 자격증명 공급자 메뉴에서 **공급자 추가** 버튼을 눌러서 EKS OIDC Provider 등록

      <img width="1354" alt="image" src="https://github.com/tedilabs/fastcampus-devops/assets/33750210/6440b3ab-f1ca-4560-b831-039fa9935a19">

## IRSA 생성

### IAM Role 생성

- 이전 단계에서 등록한 IAM Identity Provider를 Trusted Entity로 신뢰하는 IAM Role 생성
  - Condition 필드를 통해 해당 IAM Role을 사용할 수 있는 서비스 어카운트 제한
- IAM Role에 부여하는 정책(Policy)는 파드가 사용하고자 하는 AWS 권한 정책 부여

<img width="1506" alt="image" src="https://github.com/tedilabs/fastcampus-devops/assets/33750210/b81285a1-1401-467b-8325-300ffd5485e8">

### 테라폼 모듈 구성

- 테라폼 리소스 문서 - [aws_iam_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)
- [테라폼 레지스트리(모듈)](https://registry.terraform.io/modules/tedilabs/container/aws/latest/submodules/eks-irsa)
- [Github 레포지토리](https://github.com/tedilabs/terraform-aws-container/tree/v0.13.0/modules/eks-irsa)
- 리소스 구성
  - IAM Role - Service Account를 위한 IAM Role 생성

## IRSA 사용

### ServiceAccount 생성 및 Pod에 연결하기

- ServiceAccount를 생성하여 annotation을 통해 IAM Role ARN 명시
- Pod는 해당 ServiceAccount와 연결
- IAM Role 생성 시에 Condition에 추가했던 네임스페이스/서비스 어카운트와  동일하게 생성할 것
