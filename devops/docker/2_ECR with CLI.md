# ECR & ECS with CLI



## ECR 만들기 - CLI

```shell
$ aws ecr create-repository --repository-name <repository name>
```



## Distribute docker image to ECR

```shell
$ docker login --username AWS -p $(aws ecr get-login-password) <ECR URI>
$ docker tag <image_name> <ECR URI>
$ docker push <ECR URI>
```



## Create Cluster - CLI

Reference : https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_AWSCLI_Fargate.html

```shell
$ aws ecs create-cluster --cluster-name <cluster name>
```



## Create Task Definitions - CLI

1. Create `{task definition}.json` file

   ```json
   {
       "family": "sample-fargate", 
       "networkMode": "awsvpc",
     	"executionRoleArn":"ecsTaskExecutionRole",
       "containerDefinitions": [
           {
               "name": "fargate-app", 
               "image": <ECR Repository URI>, 
               "portMappings": [
                   {
                       "containerPort": 80, 
                       "hostPort": 80, 
                       "protocol": "tcp"
                   }
               ], 
               "essential": true, 
               "entryPoint": [
                 "sh",
                 "-c"
               ],
           }
       ], 
       "requiresCompatibilities": [
           "FARGATE"
       ], 
       "cpu": "256", 
       "memory": "512"
   }
   ```

   

2. Command

```shell
$ aws ecs register-task-definition --cli-input-json file://<task-definition.json path>
```



3. List Task Definitions

```shell
$ aws ecs list-task-definitions
```



## Create service

### Command

```shell
$ aws ecs create-service --cluster <cluster name> --service-name <service name> --task-definition <task definition name> --desired-count 1 --launch-type "FARGATE" --network-configuration "awsvpcConfiguration={subnets=[<public subnet>],securityGroups=[<SG>],assignPublicIp=ENABLED}"
```

### Example

```shell
$ aws ecs create-service --cluster django-cluster --service-name django-service --task-definition fargate-cli --desired-count 1 --launch-type "FARGATE" --network-configuration "awsvpcConfiguration={subnets=[subnet-098c4a373e7c939fc],securityGroups=[sg-02dfdea6f74bd494e],assignPublicIp=ENABLED}"
```

### List services

```shell
$ aws ecs list-services --cluster <cluster name>
```

### Describe the running service

```shell
$ aws ecs describe-services --cluster <cluster name> --services <service name>
```



## Clean up

### Delete the cluster

```shell
$ aws ecs delete-service --cluster fargate-cluster --service fargate-service --force
```

### Delete the service

```shell
$ aws ecs delete-cluster --cluster fargate-cluster
```