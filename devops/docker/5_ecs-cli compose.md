# ECS-cli compose



## Run Task

```shell
$ ecs-cli compose --file docker-compose.yml --ecs-params ecs-params.yml service create --launch-type FARGATE -c <cluster name>
$ ecs-cli compose --file docker-compose.yml service start -c <cluster name>
```
