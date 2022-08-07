# ECS CLI Configuration



## 다음 명령을 사용해 `profile_name`을 원하는 프로필 이름으로 바꾸고 `$AWS_ACCESS_KEY_ID` 및 `$AWS_SECRET_ACCESS_KEY` 환경 변수를 AWS 자격 증명으로 바꿔 CLI 프로필을 설정합니다.

```shell
$ ecs-cli configure profile --profile-name <profile_name> --access-key <$AWS_ACCESS_KEY_ID> --secret-key <$AWS_SECRET_ACCESS_KEY>
```



## 다음 명령을 사용하여 `launch_type`은 기본적으로 사용할 태스크 시작 유형으로, `region_name`은 원하는 AWS 리전으로, `cluster_name`은 사용할 기존 Amazon ECS 클러스터 또는 새 클러스터의 이름으로, `configuration_name`은 이 구성에 부여하고 싶은 이름으로 바꿔 구성을 완료해야 합니다.

```shell
$ ecs-cli configure --cluster <cluster_name> --default-launch-type <launch_type> --region <region_name> --config-name <configuration_name>
```
