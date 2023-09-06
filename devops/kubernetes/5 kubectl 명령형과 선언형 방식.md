# Kubectl 명령형과 선언형 방식

## 명령형 vs 선언형

### 명령형(Imperative)

- 수행하고자 하는 액션을 지시
- 적은 리소스에 대해서 빠르게 처리 가능
- 여러 명령어를 알아야 함

### 선언형(Declarative)

- 도달하고자 하는 상태를 선언
- 코드로 관리 가능 => GitOps 활용 가능
  - 변경사항에 대한 감사(Audit) 용이
  - 코드리뷰를 통한 협업
- 멱등성 보장(apply)
- 많은 리소스에 대해서도 매니페스트 관리 방법에 따라 빠르게 처리 가능
- 알아야 할 명령어 수가 적음



## kubectl 명령형 명령어

- `kubectl run -it ubuntu --image=ubuntu:focal bash` : unbuntu:focal 이미지로 unbuntu 파드 생성 (bash 명령어)
- `kubectl expose deployment grafana --type=NodePort --port=80 --target-port=3000` : grafana Deployment 오브젝트에 대해 NodePort 타입의 Service 오브젝트 생성(노드에 포트 개방)
- `kubectl set image deployment/frontend www=image:v2` : frontend Depolyment www 컨테이너 이미지를 image:v2로 변경
- `kubectl rollout undo deployment/frontend --to-revision=2` : frontend Deployment 리비전 2로 롤백



## kubectl 선언형 명령어

- `kubectl apply -f deployment.yaml` : deployment.yaml에 정의된 쿠버네티스 오브젝트 클러스터에 반영
- `kubectl delete -f deployment.yaml` : deployment.yaml에 정의된 쿠버네티스 오브젝트 제거
- `kubectl apply -k ./` : 현재 디렉토리의 kustomization.yaml 파일을 쿠버네티스 오브젝트 클러스터에 반영
