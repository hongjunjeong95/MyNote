# kubectl을 이용하여 EKS 클러스터 연결

## 사전 준비

1. AWS CLI 및 kubectl 설치
2. AWS CLI 인증 설정
   1. EKS 클러스터 생성하였던 사용자로 인증정보를 설정해야 한다.

## EKS 클러스터 연결

### ~/.kube/config 파일 내 클러스터 연결 정보 추가

- update-kubeconfig 명령어를 사용하면 자동으로 연결 정보를 추가할 수 있다.
- alias는 클러스터 연결정보의 context 이름으로 사용된다.

```sh
aws eks update-kubeconfig --region=ap-northeast-2 --name=fastcampus --alias=fastcampus
```

### 클러스터 연결 확인

```sh
kubectl cluster-info
```

