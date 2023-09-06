# 파드의 배치 전략 - Taint and Toleration

## Taint and Toleration

### 얼룩(Taint) 노드에 설정

노드에 Taint를 설정하여 임의의 파드가 할당되는 것을 방지

<img width="826" alt="image" src="https://github.com/tedilabs/fastcampus-devops/assets/33750210/881775d1-74be-4abc-986c-5f4152704544">

### 용인(Toleration) 파드에 설정

특정 Taint를 용인할 수 있는 Toleration 설정을 가진 파드는 해당 노드에 할당 가능



## 노드 Taint 관리

- 노드를 새로 구성할 때 kubelet 옵션을 통해서 기본 Taint 설정도 가능
- kubectl을 통해 노드의 Taint 관리 가능
- Label / Annotation과 비슷하지만, 추가적으로 Effect 파라미터를 가짐
  - `Key = Value:Effect`

### Effect

Taint가 노드에 설정될 시 적용될 효과

- NoSchedule - 파드를 스케줄링하지 않음
- NoExecute - 파드의 실행을 허용하지 않음
- PreferNoSchedule - 파드 스케줄링을 선호하지 않음

### commands

```sh
// minikube-mo2 노드에 role=system:Noschedule Taint 추가
kubectl taint node minikube-m02 role=system:Noschedule

// minikube-mo2 노드에 role=system:Noschedule Taint 제거
kubectl taint node minikube-m02 role:Noschedule-

// minikube-mo2 노드에 role 키를 가진 모든 Taint 제거
kubectl taint node minikube-m02 role-
```



## 다양한 Toleration 설정 방법

### 모든 종류의 Taint를 용인

```yaml
spec:
	containers:
		- name : nginx
			image: nginxdemos/hello:plain-text
			ports:
			- name: http
				containerPort: 80
				protocol: TCP
  tolerations:
  - operator: Exists
```

### 키가 role인 모든 Taint를 용인

```yaml
spec:
	containers:
		- name : nginx
			image: nginxdemos/hello:plain-text
			ports:
			- name: http
				containerPort: 80
				protocol: TCP
  tolerations:
  - key: role
  	operator: Exists
```

### 키가 role이고 효과가 NoExecute인 모든 Taint 용인

```yaml
spec:
	containers:
		- name : nginx
			image: nginxdemos/hello:plain-text
			ports:
			- name: http
				containerPort: 80
				protocol: TCP
  tolerations:
  - key: role
  	operator: Exists
  	effect: NoExecute
```

### role=system:Noschedule Taint를 용인

```yaml
spec:
	containers:
		- name : nginx
			image: nginxdemos/hello:plain-text
			ports:
			- name: http
				containerPort: 80
				protocol: TCP
  tolerations:
  - key: role
  	operator: Equal
  	value: system
  	effect: NoSchedule
```

