# Patches

## Patches Strategic Merge

### kustomize.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../base

namePrefix: dev-

patchesStrategicMerge:
- resources.yaml
- service.yaml
```

- resources : `../base` directory를 kustomize로 사용한다.
- namePrefix : 전체 resource에 대해서 `dev-`를 붙인다.
- patchesStrategicMerge : 전략적 머지 기법을 사용



### resources.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  template:
    spec:
      containers:
      - name: nginx
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
```

디플로이먼트 객체를 만들기 위해서는 파드 템플릿을 만들 수 있는 selector과 containers의 image 값도 명시되어 있어야 하지만 그렇지 않다. 그 이유는 디플로이먼트 객체를 생성하기 위함이 아니라 patch로 이용되기 때문이다. patch는 수정을 필요로 하는 값만 변경한다. 수정을 하기 위해서는 target을 찾아야 하는데,

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
```

위 3가지 값만 명시를 해주면 kustomize가 strategyMerge를 수행할 때 어떤 오브젝트를 수정할지 타켓을 찾아낼 수 있다. spec으로 쓰여져 있는 내용 밑은 패치 내용이다. 계속 따라가서 `name: nginx`에 `resources` 값을 추가하라는 의미다. 



### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  type: NodePort
```



## Patches Json 6902

 JSON6902라는 것은 RFC 6902가 JSON Patch에 대한 기술 넘버다.

Ref : https://datatracker.ietf.org/doc/html/rfc6902

### base/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      name: hello
      labels:
        app: hello
    spec:
      containers:
      - name: nginx
        image: nginxdemos/hello:plain-text
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
```



### kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../base

namePrefix: dev-

patchesJson6902:
- target:
    version: v1
    kind: Deployment
    name: hello
  path: resources.yaml
- target:
    version: v1
    kind: Service
    name: hello
  path: service.yaml
```



### resources.yaml

```yaml
- op: add
  path: /spec/template/spec/containers/0/resources
  value:
    requests:
      cpu: 100m
      memory: 64Mi
```

- op는 operator의 약자다. add, remove, replace, move, copy와 test 등이 있다.
- add는 path에 해당하는 곳에 value를 추가하라는 뜻이다.
