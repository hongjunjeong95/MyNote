# API Resource - ReplicaSet

## 레플리카셋이란?

<img width="756" alt="image" src="https://user-images.githubusercontent.com/33750210/233323420-ad79de4a-a833-41f3-9155-653d2b56d9f5.png" style="zoom:50%;" >

- 파드의 수를 늘리고 싶다면? (Scale-out)

### 레플리카셋(ReplicaSet)

- 정해진 수의 파드가 항상 실행될 수 있도록 관리
- 기존 실행중이던 파드에 문제가 생기면 파드를 다시 스케줄링
- ReplicationController의 신규 버전 (현재는 Deprecated)

파드와 마찬가지로 레플리카셋을 직접 관리하는 경우가 거의 없음!



## 레플리카셋의 동작원리

- ReplicaSet Controller가 Control Plane에 존재.
- spec.selector에 대응되는 파드의 수가 spec.replicas와 동일한지 지속적으로 검사하고, 다를 경우 스케일 아웃 혹은 인 진행

<img width="376" alt="image" src="https://user-images.githubusercontent.com/33750210/233884124-50428e63-4baa-4b2e-9499-df7d701a955a.png">

### 레이블 셀렉터(Label Selector)

- 쿠버네티스 오브젝트는 모두 metadata.labels에 Key - Value 형태의 레이블 값을 가짐
- 특정 오브젝트 목록을 필터링하기 위한 기능이 Label Selector
- matchLabels와 matchExpressions 옵션 제공
- <strong style="color:#eb4444">많은 쿠버네티스 API 리소스가 Label Selector을 통해 기능을 제공한다. => 리소스 간 느슨한 결합 유지 (Loosely Coupled)</strong>

### ReplicaSet의 옵션

spec.selector에 대응되는 파드의 수가 spec.replicas와 동일한지 지속적으로 검사하고, 다를 경우 스케일 아웃 혹은 인 진행. 하지만 파드가 레플리카셋에서 관리하는 파드인지는 검사하지는 않는다.  

<img width="179" alt="image" src="https://user-images.githubusercontent.com/33750210/233884390-bfa51b90-a539-43bb-b293-f4416cad0667.png">

- app - hello를 레이블을 가지고 있는 pod의 수가 3이 되도록 유지시켜준다.

### Template Options

<img width="381" alt="image" src="https://user-images.githubusercontent.com/33750210/233884430-21579bdf-ba81-4d60-96d6-3d8f8a5164c7.png">

ReplicaSet이 만드는 pod 템플 



