# Kustomize를 이용한 쿠버네티스 매니페스트 관리

**Ref**

* [kustomize](https://kustomize.io/)
* [kustomize github](https://github.com/kubernetes-sigs/kustomize)
* [kustomize cli](https://kubectl.docs.kubernetes.io/references/kustomize/)

## Kustomize 소개

쿠버네티스 매니페스트 파일을 효율적으로 관리하기위해 만들어진 오픈소스 도구
원본 YAML 파일을 보존한 채로 목적에 따라 변경본을 만들어 사용할 수 있도록 하는 것을 목표로 함



## kustomization.yaml

Kustomize가 사용하는 YAML 매니페스트 파일

### Base Manifests

Kustomization에 의해 참조되는 Kustomization
보통 기본 설정으로 구성된 쿠버네티스 매니페스트 묶음

### Overlay Manifests

Base Manifests에 변형을 가하기 위해 사용되는 Kustomization Overaly도 다른 누군가의 Base가 될 수 있음

### 구조

<img width="880" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/b9c622e9-6705-42d6-84f8-1f8587558a63">

kustomization.yaml 파일에서 resource로 deployment.yaml을 선택하고 overlay로 merge시킬 overlay.yaml을 선택한다. 그러면 kind와 metadata를 필터삼아서 overlay.yaml파일이 deployment.yaml의 리소스를 찾아서 `replicas:10`을 덮어씌운다.

만약 images 속성으로 이미지를 바꾸게 되 두 가지 목적으로 사용된다.

- 이미지 레지스트리 위치 변경

- 이미지 버전 변경

  ```yaml
  images:
  # grafana/grafana 이미지를 찾아서 newTag 속성의 버전 "8.2.2"로 바꾼다.
  - name: grafana/grafana 
  	newTag: "8.2.2"
  # nginxdemos/hello 이미지를 찾아서 nginx 이미지로 바꾼다.
  - name: nginxdemos/hello
    newName: nginx
    newTag: "latest"
  ```



## Kustomize 주요 명령어

현재 디렉토리에 kustomization.yaml  파일 생성

```
kustomize create
```

현재 디렉토리에 kustomization.yaml 파일 해석하여 쿠버네티스 매니페스트 출력

```
kustomize build .
```

URL을 통해 원격에 위치한 kustomization.yaml 파일 해석하여 쿠버네티스 매티페스트 출력

```
kustomize build https://github.com/tedilabs/k8s-repository/observability/grafana/v8.2
```

kustomization.yaml 파일 내용 클러스터에 적용

```
kustomize build . | kubectl apply -f -
```

kustomization.yaml 파일 내용 클러스터에서 삭제

```
kustomize build . | kubectl delete -f -
```



### kubectl 통합

- Kustomize가 인기를 얻으면서 Kubernetes 버전 1.14에서 kubectl 내에 kustomize 버전 v2.0.3을 내장
- 하지만, 쿠버네티스 버전 1.20까지는 관리가 되지 않아 kustomize 버전 v2.0.3 유지
- 2021-12 기준 kustomize 최신 버전은 v4.4.1

```
# kustomize build .
kubectl kustomize .

# kustomize build . | kubectl apply -f -
kubectl apply -k .

# kustomize build . | kubectl delete -f -
kustomize delete -k .
```

<img width="643" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/195d630f-25e0-4f09-93d1-3981c1e1fcc7" style="zoom:67%;" >

