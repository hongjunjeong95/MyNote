# API Resources

## API 리소스와 오브젝트

<img width="1272" alt="image" src="https://user-images.githubusercontent.com/33750210/232384507-4e809062-9e60-4208-8c24-d0b347bf2ff8.png" style="zoom:60%;" >

### API 리소스

- 쿠버네티스가 관리할 수 있는 오브젝트의 종류
- Pod, Service, ConfigMap, Secret, Node, ServiceAccount, Role

### Object

- API 리소스를 인스턴스화 한 것

### 명령어

- `kubectl api-resources` : 현재 쿠버네티스 클러스터가 지원하는 API 리소스 목록 출력
- `kubectl explain pod` : 특정 API 리소스에 대해 간단한 설명 확인 

## 매니페스트 파일

쿠버네티스는 오브젝트를 YAML 기반 매니페스트 파일로 관리

<img width="846" alt="image" src="https://user-images.githubusercontent.com/33750210/232650371-eb07419c-bc8a-4c5a-a189-98b9fb942751.png" style="zoom:50%;" >

- apiVersion : 오브젝트가 어떤 API 그룹에 속하고 API 버전이 몇인가?
- kind : 오브젝트가 어떤 API 리소스인가?
- metadata : 오브젝트를 식별하기 위한 정보(이름, 네임스페이스, 레이블 등)?
- spec : 오브젝트가 가지고자 하는 데이터는?
  - API 리소스에 따라 spec 대신 data, rules, subjects 등 다른 속성 사용
  - data : configMap, secret은 spect 대신 data를 사용
  - rules : role은 rules를 사용

## Labels와 Annotations

- 모든 쿠버네티스 오브젝트는 Labels와 Annotations 메타데이터를 가질 수 있음
- Optional 함
- 둘 모두 문자열(String) 형식의 Key - Value 데이터를 기록

### Labels

- 오브젝트를 식별하기 위한 목적
- 검색 / 분류 / 필터링 등의 목적으로 사용
- 쿠버네티스 내부 여러 기능에서 Label Selector 기능 제공
- Annotations와 다른 점은 좀 더 정확한 리소스 이름을 정의한다던가 종류가 무엇인지 혹은 어떤 팀의 소유인지, 소유자가 누구인지 등의 **식별** 목적의 데이터다.

### Annotations

- 식별이 아닌 다른 목적으로 사용
- 사람이 읽기 위한 목적 보다는 보통 쿠버네티스의 애드온이 해당 오브젝트를 어떻게 처리할지 결정하기 위한 설정 용도로 사용
