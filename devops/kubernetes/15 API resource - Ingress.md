# API Resource - Ingress

## Ingress란?

외부로부터의 요청에 대해 TLS 설정 관리 및 라우팅 관리와 L4 레벨이 아닌 L7 레벨에서 외부 요청을 처리할 때 사용한다.
게이트웨이 같은 존재라고 생각하면 되는데, 요청을 받으면 Post나 Put에 따라 요청을 서비스에 분산시킬 수 있다. 이 요청을 받은 서비스는 TCP/IP에 따라 Pod에 요청을 분산한다.

- 외부 요청을 받아 L7 (어플리케이션 레이어)에서 어떻게 처리할 것인지를 결정
- 라우팅 기능 수행 (Host 단위, Path 단위로 라우팅 가능)
- SSL / TLS 통신 암호화 처리 : 각 연결 호스트에 대한 인증서 적용

<img width="919" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/04657f8d-2eb1-4478-9e24-f1bc053db0a5" style="zoom:50%;" >



## 인그레스 컨트롤러와  인그레스 클래스

쿠버네티스 클러스터는 기본적으로 Ingress API 리소스를 다루는 Ingress Controller를 제공하지 않음
Ingress API 리소스에 대한 스펙만 제공
=> 사용자가 직접 원하는 Ingress Controller 설치 필요

### 대표적인 Ingress Controller

- **Nginx Ingress Controller**(minikube에 기본적으로 포함되어 있음)
- Kong Ingress Controller
- **AWS Load Balancer Controller**
- Google Load Balancer Controller

### 인그레스 클래스

하나의 클러스터에서 여러 인그레스 컨트롤러를 사용할 수 있도록 하기 위해 만들어진 리소스
**IngressClass = IngressController + Configuration**

### 인그레스

- 라우팅 규칙 및 TLS 설정을 정의
- 하나의 인그레스 클래스와 연결



## NGINX Ingress Controller

https://github.com/kubernetes/ingress-nginx/

Nginx 웹 서버 기반의 Ingress Controller

<img width="1022" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/db22a620-2430-446d-918b-f4b3a7cb3784" style="zoom:65%;" >



## AWS Load Balancer Controller

https://github.com/kubernetes-sigs/aws-load-balancer-controller

AWS에서 관리하는 오픈소스 컨트롤러로 다음의 기능을 제공

- AWS ALB(Application LoadBalancer) 기반의 Ingress Controller
- AWS NLB(Network LoadBalancer) 기반의 LoadBalancer 타입 Service

<img width="1397" alt="image" src="https://github.com/huggingface/transformers/assets/33750210/9f41aa6f-9870-4743-91bf-d5f7e466f5ef">



## 예제

### ingress-path.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /hello
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              name: http
  - http:
      paths:
      - path: /grafana
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              name: http
```

{IP}/{path}로 요청을 보낸다.

### ingress-host.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: hello.fastcampus
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              name: http
  - host: grafana.fastcampus
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              name: http
```

{IP}와 Header에 Host에 값을 넣어서 요청을 보낸다.

### ingress-default-backend.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default-backend
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  defaultBackend:
    service:
      name: httpd
      port:
        number: 80
  rules:
  - host: hello.fastcampus
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              name: http
  - host: grafana.fastcampus
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              name: http
```

- {IP}와 Header에 Host에 값을 넣어서 요청을 보낸다.
- `defaultBackend`는 rules에 정의하지 않은 곳으로 요청이 들어오면 처리하는 백엔드다.
