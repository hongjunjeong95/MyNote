# API Resource - Job과 CronJob

## Job이란?

- 특정 동작을 수행하고 종료하는 작업을 정의하기 위한 리소스
- 내부적으로 파드를 생성하여 작업 수행
- Pod의 상태가 Running이 아닌 Completed가 되는 것이 최종 상태
- 실패시 재시작 옵션, 작업 수행 회수, 동시 실행 수 등 세부 옵션 제공

### job.yaml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: hello
        image: ubuntu:focal
        command: ["sh", "-c", "echo Hello World!"]
```

- 쿠버네티스의 batch api 그룹에 속하기 때문에 `batch/v1`으로 표기한다.
- Job을 만들 때 반드시 `restartPolicy`를 명시해줘야 한다.
  - Always, Never, OnFailure
  - 계속해서 실행되는 것이 아니라 completed가 되면 종료되는 오브젝트다. 그래서 Always 옵션은 부적절하다.
- Deployment를 사용할 때는 Label selector를 통해가지고 디플로이먼트 객체가 어떤 파드 오브젝트를 관리하는지 연결시켜준다. 하지만 job과 같은 부분은 selector가 빠진 부분을 볼 수 있다. Job을 만들 때 Job Controller가 기본적으로 pod 템플릿에 있는 pod와 기본적인 레이블 셀렉터를 임의로 생성해서 넣어준다. 

### job-parallelism.yaml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  completions: 10
  parallelism: 2
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: hello
        image: ubuntu:focal
        command: ["sh", "-c", "sleep 2; echo Hello World!"]
```

- completions : Job을 몇 번 성공시킬 것인가
- paralleism : 최대 동시 실행 횟수

### job-deadline.yaml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  activeDeadlineSeconds: 3
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: hello
        image: ubuntu:focal
        command: ["sh", "-c", "sleep 5; echo Hello World!"]
```

- activeDeadlineSeconds : Job이 최대 실행될 수 있는 타임아웃이다. 위 예제에서는 3초를 넘어서면 실패로 간주한다.

## CronJob이란?

- 주기적으로 특정 동작을 수행하고 종료하는 작업을 정의하기 위한 리소스
- 리눅스의 Cron 스케줄링 방법을 그대로 사용
- 내부적으로 잡을 생성하여 작업 수행
- 주기적으로 데이터를 백업하거나 데이터 점검 및 알림 전송 등의 목적으로 사용

<img width="443" alt="image" src="https://user-images.githubusercontent.com/33750210/235811248-f841daac-097b-4108-820c-2a51626818d4.png" style="zoom:50%;" >

### cronjob.yaml

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-every-minute
spec:
  schedule: "*/1 * * * *"
  successfulJobsHistoryLimit: 5
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: hello
            image: ubuntu:focal
            command: ["sh", "-c", "echo Hello $(date)!"]
```

- jobTemplate에서처럼 Job에서 사용한 스펙이 그대로 왔다고 보면 된다.
- successfulJobHistoryLimit : 성공한 job의 기록을 최대 몇 개를 남길 것인가
