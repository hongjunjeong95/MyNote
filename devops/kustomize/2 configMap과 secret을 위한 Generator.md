# ConfigMap과 Secret을 위한 Generator

## kustomize.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml

configMapGenerator:
- name: mysql-config
  literals:
  - MYSQL_DATABASE=devops
  envs:
  - mysql.env
- name: test-files
  files:
  - files/test1.txt
  - test2.txt=files/test2.txt

secretGenerator:
- name: mysql-secret
  literals:
  - MYSQL_ROOT_PASSWORD=fastcampus

# These labels are added to all configmaps and secrets.
generatorOptions:
  labels:
    env: dev
  annotations:
    managed_by: kustomize
  # disableNameSuffixHash is true disables the default behavior of adding a
  # suffix to the names of generated resources that is a hash of
  # the resource contents.
  # disableNameSuffixHash: true
```

### configMapGenerator

쿠버네티스의 컨피그맵을 제공한다.

- literals : key-value 형식으로 데이터를 제공
- envs : env 파일을 읽어서 환경변수 제공
- files
  - `files/test1.txt` 같이 value만 지정을 하면 파일 경로 자체가 키가 된다.
  - `test2.txt=files/test2.txt` : test2.txt가 키가 되고 files/test2.txt가 value가 된다.

### secretGenerator

쿠버네티스의 시크릿을 제공한다.

- literals : key-value 형식으로 데이터를 제공
- envs : env 파일을 읽어서 환경변수 제공
- files
  - `files/test1.txt` 같이 value만 지정을 하면 파일 경로 자체가 키가 된다.
  - `test2.txt=files/test2.txt` : test2.txt가 키가 되고 files/test2.txt가 value가 된다.

### generatorOptions

config & secret에 대해서 일반적인 옵션을 제공한다.

- labels
- annotations
- disableNameSuffixHash : 디폴트로 true인데 뒤에 해시를 생성하는 것을 원치 않는다면 false로 바꾼다. 

Generator로 생성되는 키값들은 뒤에 hash가 붙는다. Generator가 config & secret이 출력되는 결과물의 형태에 따라서 해시값이 변경된다. 해시를 추가하는 이유는 디플로이먼트라는 객체는 매니페스트 내용이 변경되지 않는 이상 파드를 재시작 하지 않는다. 그래서 config & secret이 변경되더라도 디플로이먼트는 알 수가 없어서 재시작되지 않는다. 해시를 추가하게 된다면 변경됨을 알게 되어 파드가 재생성될 수 있다. 하지만 이것이 항상 장점은 아니기 때문에 신중하게 결정해야 한다. 
