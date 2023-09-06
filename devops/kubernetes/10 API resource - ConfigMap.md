# API Resource - ConfigMap

## ConfigMap이란?

- 어플리케이션의 설정 값을 컨테이너에 주입할 때 사용한다. Secret과 달리 민감하지 않고 어플리케이션에 설정을 위한 정보들만 주입한다.
- 설정 정보를 환경변수 혹은 볼륨의 형태로 파드에 전달하기 위한 목적으로 사용
- 파드에서 직접 환경변수를 관리하지 않고 ConfigMap을 분리하여 목적에 따라 설정 데이터를 다르게 주입 가능

<img width="581" alt="image" src="https://user-images.githubusercontent.com/33750210/234157358-a460d0fb-3f71-4ef6-9f8f-9478072a2a42.png" style="zoom:50%;" >



## ConfigMap의 사용 방법

ConfigMap은 파드 내 컨테이너의 환경변수 혹은 볼륨으로 연결 가능하다. 두 가지 방법이 있는데

- ConfigMap의 값을 컨테이너의 환경변수로 사용
- ConfigMap의 값을 파드 볼륨으로 마운트하여 사용

### No

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      name: mysql
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: fastcampus
        - name: MYSQL_DATABASE
          value: devops
        ports:
        - name: http
          containerPort: 3306
          protocol: TCP
```

configmap을 사용하지 않으면 일일이 spec.env에 입력을 해줘야 한다.

### env-from

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      name: mysql
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        envFrom:
        - configMapRef:
            name: mysql-config
```

spec.envFrom.configMapRef에서 이름이 mysql-config인 ConfigMap을 연결한다.

### env-value-from

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      name: mysql
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            configMapKeyRef:
              name: mysql-config
              key: MYSQL_ROOT_PASSWORD
```

spec.env.valueFrom.configMapRef에서 이름이 env의 key 는 MYSQL_ROOT_PASSWORD로 하고 value는 `valueFrom.configMapKeyRef.name(my-config)`에서 `valueFrom.configMapKeyRef.key(MYSQL_ROOT_PASSWORD)`를 찾아서 주입한다.

### volume

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      name: mysql
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        envFrom:
        - configMapRef:
            name: mysql-config
        volumeMounts:
        - mountPath: /tmp/config
          name: mysql-config
      volumes:
      - name: mysql-config
        configMap:
          name: mysql-config
```

volumes 목록은 해당 파드가 사용하게될 볼륨 목록을 정의한다. volumeMounts에서는 `/tmp/config`라는 경로로 `mysql-config`를 볼륨에 마운트 하라는 뜻이다.

- configMap은 directory로 마운되게 된다. 하위에 있는 각각의 key-value들이 file-data로 만들어진다.
- `/tmp/config` 디렉토리에 가게되면 mysql-config에 설정된 keys가 파일들로 생성되어 있다. cat 명령어를 이용해서 파일을 살펴보면 value 값을 확인할 수 있다.



## kubectl ConfigMap 생성 명령어

- `kubectl crate configmap my-config` : my-config 이름의 ConfigMap 생성
- `kubectl create configmap my-config --from-file config.yaml` : my-config 이름의 ConfigMap 생성 - 로컬의 config.yaml 파일을 config.yaml을 키로 저장
- `kubectl create configmap my-config --from-file config=config.yaml` : my-config 이름의 ConfigMap 생성 - 로컬의 config.yaml 파일을 config을 키로 저장
- `kubectl create configmap my-config --from-file config=config.yaml --dry-run -o yaml` : my-config 이름의 ConfigMap YAML 출력 - 로컬의 config.yaml 파일을 config을 키로 저장
