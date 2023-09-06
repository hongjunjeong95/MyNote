# 파드의 배치 전략 - Affinity

## nodeAffinity을 이용한 배치

- 선호하는 노드를 설정하는 방법으로, nodeSelector 보다 확장된 Label Selector 기능 지원
- matchExpressions 사용 가능 (In, NotIn, Exists, DoesNotExist, Gt, Lt 등)
- 여러 유즈케이스에 활용 가능한 다양한 옵션 제공
  - 반드시 충족해야 하는 조건 **(Hard)**
    - **required**DuringScheduling**Ignored**DuringExecution
  - 선호하는 조건 **(Soft)**
    - **preferred**DuringScheduling**Ignored**DuringExecution
- <strong style="color:#eb4444">IgnoredDuringExecution => 실행중인 워크로드에 대해서는 해당 규칙을 무시한다.</strong>



## podAffinity을 이용한 배치

선호하는 파드(클러스터에서 이미 실행중인 파드)를 설정하는 방법으로, 사용법은 nodeAffinity와 거의 동일 여러 유즈케이스에 활용 가능한 다양한 옵션 제공

- 반드시 충족해야 하는 조건 **(Hard)**
  - **required**DuringScheduling**Ignored**DuringExeution**
- 선호하는 조건 **(Soft)**
  - **preferred**DuringScheduling**Ignored**DuringExecution

**토폴로지 키** **(Topology Key)**

Label Selector를 수행할 노드의 범위를 결정

- 노드 단위: kubernetes.io/hostname
  - 노드를 띄우면 기본 설정
- 존 단위: topology.kubernetes.io/zone
  - AWS EKS에서 띄울경우 AZ 존을 의미
- 리전 단위: topology.kubernetes.io/region
  - AWS EKS에서 서울 리전 등을 의미

```yaml
spec:
  containers:
  - name: nginx
    image: nginxdemos/hello:plain-text
    ports:
    - name: http
      containerPort: 80
      protocol: TCP
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - mysql
        topologyKey: kubernetes.io/hostname
```

동일한 노드(kubernetes.io/hostname) 안에서 key(app)은 values(mysql)이라는 레이블을 가진 파드를 선호해서 그 안에 띄운다.

## podAntiAffinity을 이용한 배치

선호하지 않는 파드를 설정하는 방법으로, podAffinity를 podAntiAffinity로만 변경하면 사용법 동일 여러 유즈케이스에 활용 가능한 다양한 옵션 제공

- 반드시 충족해야 하는 조건 **(Hard)
  * required**DuringScheduling**Ignored**Duringxecution**
- 선호하는 조건 **(Soft)
  - **preferred**DuringScheduling**Ignored**DuringExecution

**토폴로지 키 (Topology Key)**

Label Selector를 수행할 노드의 범위를 결정

- 노드 단위: kubernetes.io/hostname
- 존 단위: topology.kubernetes.io/zone
- 리전 단위: topology.kubernetes.io/region
