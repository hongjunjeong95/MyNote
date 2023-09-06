# API Resource - Secret

## Secret이란?

패스워드, API Key, SSH key 등 민감한 정보를 컨테이너에 주입해야 할 때 쓰인다.

<img width="1148" alt="image" src="https://user-images.githubusercontent.com/33750210/234194289-044b9edb-6f5c-4453-87d6-cc9fe57daffa.png" style="zoom:70%;" >

- ConfigMap과 사용법 비슷
- 시크릿은 사용 목적에 따라 몇 가지 종류로 나누어 짐
- 쿠버네티스는 기본적으로 시크릿 값을 저장할 때 Base64 인코딩. 그래서 접근만 된다면 값을 읽을 수 있다. AWS EKS같은 경우에는 KMS를 이용해서 encrypt를 한 번 더 진행하기도 한다.



## Secret의 종류

1. **Opaque(generic) - 일반적인 용도의 시크릿**
2. dockerconfigjson - 도커 이미지 저장소 인증 정보
3. tls - TLS 인증서 정보
4. service-account-token - ServiceAccount의 인증 정보



## kubectl Secret 생성 명령어 (generic)

- `kubectl create secret generic my-secret` : my-secret 이름의 generic 타입 Secret 생성
- `kubectl create secret generic my-secret --from-file secret.yaml` : my-secret 이름의 generic 타입 Secret 생성 - 로컬의 secret.yaml 파일을 secret.yaml을 킬로 저장
- `kubectl create secret generic my-secret --from-file secret=secret.yaml` : my-secret 이름의 generic 타입 Secret 생성 - 로컬의 secret.yaml 파일을 secret을 키로 저장
- `kubectl create secret generic my-secret --from-file secret=secret.yaml --dry=run -o yaml` : my-secret 이름의 generic 타입 Secret YAML 출력 - 로컬의 secret.yaml 파일을 secret을 키로 저장



## Secret의 선언적 관리

**Secret은 Git과 같은 버전관리시스템에서 관리하기에 기밀 정보가 담겨 있어 부적절**

### External Secrets

- HashiCrop Vault, AWS Secret Manager 등과 통합
- ExternalSecret 오브젝트를 생성하면 컨트롤러가 프로바이더로부터 기밀 값을 가져와서 Secret 오브젝트 생성
- https://github.com/external-secrets/external-secrets

### Sealed Secrets

- 쿠버네티스 클러스터 상에 컨트롤러 실행
- 클러스터 상에 암호화 키 보관
- kubeseal CLI가 컨트롤러와 통신하며 데이터 암호화
- SealedSecret 오브젝트를 생성핳면 컨트롤러가 복호화하여 Secret 오브젝트 생성
- https://github.com/bitnami-labs/sealed-secrets

