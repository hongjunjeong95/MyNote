# EKS 클러스터 인증 관리

## EKS 제공 클러스터 인증 방법

### aws-auth ConfigMap

- AWS 인증 주체와 쿠버네티스 인증 주체 간의 연결 정보 설정
- AWS 인증 주체 : Root Account / IAM User / IAM Role
- 쿠버네티스 인증 주체 : User / Group

### OIDC Identity Provider 인증

- EKS 클러스터에서 OIDC Identity Provider를 연결하여 OIDC 인증 가능
- 조직 내 OIDC를 지원하는 임직원 계정계를 활용하고 있다면 계정 통합 측면에서 유리

<strong style="color:#eb4444">그 외 쿠버네티스 애드온을 통해 다른 인증방법 활용도 가능</strong>

## aws-auth ConfigMap

### Ref

- https://github.com/kubernetes-sigs/aws-iam-authenticator
- https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html

### aws-iam-authenticator

- EKS에서 AWS 인증 주체와 쿠버네티스 인증 주체 간의 매핑을 위한 컴포넌트
- 쿠버네티스 클러스터 상의 kube-system 네임스페이스 내 aws-auth ConfigMap을 설정파일로 사용
- EKS NodeGroup과 Fargate도 클러스터 연결을 위해 동일 인증방식을 사용

- update-kubeconfig 명령어를 사용하면 자동으로 연결 정보를 추가할 수 있다.
- alias는 클러스터 연결정보의 context 이름으로 사용된다.

### aws-auth ConfigMap 보기

```sh
kubectl get configmap aws-auth -n kube-system -o yaml
```

