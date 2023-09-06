# API Resource - Deployment

## 디플로이먼트란?

- 서비스 버전이 업데이트되어 파드를 새로운 버전의 이미지 파드로 교체해야 한다면?
- 새 버전에 이슈가 발견되어 롤백을 진행해야 한다면?
- 즉, 업데이트와 롤백에 과정에 관여를 하는 것이 디플로이먼트다.

<img width="756" alt="image" src="https://user-images.githubusercontent.com/33750210/233323420-ad79de4a-a833-41f3-9155-653d2b56d9f5.png" style="zoom:50%;" >

- 파드의 이미지 버전이 갱신될 때 배포 전략을 설정
- 디플로이먼트 오브젝트를 생성하면 대응되는 ReplicaSet과 Pod 자동 생성
- 기본적으로 Recreate 전략과 RollingUpdate 전략 지원
- <strong style="color:#eb4444">사용자는 특수한 목적이 아니라면 파드와 레플리카셋이 아닌 디플로이먼트로 워크로드 관리</strong>



## 디플로이먼트의 배포전략

### 재생성(Recreate)

- 기존 레플리카셋의 파드를 모두 종료 후 새 레플리카셋의 파드를 새로 생성
- 프로덕션에서 사용하기에는 주의가 필요

### Rolling Update

- 세부 설정에 따라 기존 레플리카셋에서 새 레플리카셋으로 점진적으로 이동
  - maxSurge : 업데이트 과정에 spec.replicas 수 기준 최대 새로 추가되는 파드 수
  - maxUnavailable : 업데이트 과정에 spec.replicas 수 기준 최대 이용 불가능 파드 수

#### 새 파드 생성 후 기존 파드 삭제 반복 

```
maxSurge : 1
maxUnavailable : 0
```

- 장점 : 10개의 트랙픽을 감당할 수 있다면, 파드가 줄지 않아 충분히 대응할 수 있다.
- 단점 : 클러스터의 노드 자원이 부족해질 수도 있다.

#### 기존 파드 삭제 후 새 파드 생성 반복

```
maxSurge : 0
maxUnavaliable : 1
```

- 장점 : 노드 자원 초과될 일이 없음
- 단점 : 트래픽이 많을 경우 자원이 부족해 대응을 못 할 수도 있다.

#### 예시

<img width="364" alt="image" src="https://user-images.githubusercontent.com/33750210/233888846-34d5c160-874f-4da5-b3bd-9f7c2ac8f4f4.png">

- strategy에서 type을 **RollingUpdate**로 해주지 않으면 default로 recreate로 된다.
- minReadySeconds : 새로운 파드가 띄워지고 나서 5초 동안 ready하는 시간을 가지라는 뜻이다.
- revisionHistoryLimit : 디플로이먼트가 새로운 레플리카를 만들면서 이미지 버전이 업데이트 될 때마다 revision을 만든다. 이 때 revision을 최대 몇 개까지 가지고 있을 것인가다.
  - `kubectl rollout history deployment rolling` : rolling deployment object에 revision history를 확인할 수 있다. 기본적으로 kubectl은 어떠한 change-cause로 발생한 revision인지 남기지 않는다. 이 때, record 옵션을 통해가지고 남길 수 있다.
  - `kubectl apply -f <yaml file> --record` 명령어를 사용하고 다시 `kubectl rollout history deployment rolling` 명령어를 사용하면 `CHANGE-CAUSE`에 Revision 기록이 남는다. 
  - `kubectl rollout history -f <yaml file>` 이 명령어 파일을 지정해서 history를 볼 수도 있다.
- Rollback
  - `kubectl roolout undo deployment rolling --to-revision=1` : revision 1로 롤백하는 명령어다.
  - `kubectl rollout status deployment rolling` : rollout의 상태를 확인할 수 있다.
