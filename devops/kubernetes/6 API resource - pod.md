# API Resource - pod

## 파드란?

- 쿠버네티스가 컨테이너를 다루는 기본 단위
  - 쿠버네티스에서는 컨테이너를 직접적으로 생성할 수 있는 방법은 없다. 어떻게 생성을 하든 가장 기본이 되는 단위는 pod다.
- 1개 이상의 컨테이너로 구성된 컨테이너 집합
- 동일 파드 내 컨테이너는 여러 리눅스 네임스페이스를 공유 => **네트워크 네임스페이스 공유 (동일 IP 사용)**
-  <strong style="color:#eb4444">사용자가 파드를 직접 관리하는 경우는 거의 없음</strong>

### API Resources

- Statefulset
- Deployment
- Daemonset
- Job
- Cron Job
- Replicaset

위와 같은 API 리소스를 만들어서 어플리케이션을 띄우더라도 내부적으로 Pod를 띄우게된다.

## 파드 관련 kubectl 명령어

- `kubectl get pod` : 파드 목록 확인
- `kubectl describe pod [pod name]` : 특정 파드 상태 확인
  - `kubectl describe pod hello`

- `kubectl exec -i -t [pod name] -c [container name] bash` : 특정 파드에 명령어 전달
  - `kubectl exec -i -t hello -c debug bash`
- `kubectl logs [pod name] -c [conatiner name]` : 특정 파드 로그 확인
  - `kubectl logs pod/hello -c debug`

## 멀티 컨테이너 파드와 사이드카 패턴

- 동일 파드 내 컨테이너는 모두 같은 노드에서 실행(네임스페이스를 공유하기 때문)

### 사이드카 패턴(Side-car Pattern)

- 메인 컨테이너를 보조하는 컨테이너와 같이 실행하는 구조
- 주요 유즈 케이스
  - Filebeat와 같은 로그 에이전트로 파드 로그 수집
  - Envoy와 같은 프록시 서버로 서비스메시 구성
  - Vault Agent와 같은 기밀 데이터 전달 목적
  - Nginx의 설정 리로드 역할 에이전트 구성

