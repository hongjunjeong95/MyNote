# CodeDeploy

Jenkins를 이용해서 build와 deploy를 관리하면 build용 jenkins(ec2)와 deploy용 jenkins(ec2)를 관리해야 하는 운영 cost가 발생한다. 그래서 AWS의 codePipeline, codeBuild, codeDeploy를 활용할 것이다.

## 장점

* 빌드를 담당하는 jenkins slave를 관리할 필요가 없어진다.
* CodeDeploy 자체 내에서 컨테이너를 띄워서 독립적인 환경에서 배포를 구축할 수 있음

## 기능

* Computing service(EC2, Lambda, on-Premise)에 배포 자동화 도구
* 다양한 배포 방식 지원
  * In-place deployment : 하나의 target server에 워크로드를 올리고 내림
  * Rolling deployment : 여러 개의 target server가 있을 때 하나씩 배포
  * Blue / Green deployment : Auto-scaling 같은 경우 기존의 blue version이 구동하고 있다면 green version을 새로 띄워서 트래픽을 키운다.
  * Canary deployment : 1%만 새로운 버전을 배포해보고 그것이 문제가 없으면 점점 증가하여 새로운 버전으로 대체
* Rollback 기능 지원 : 배포했을 때 문제가 생기면 롤백
* Notification 기능 지원(AWS SNS)
* Continuous Delivery 기능 지원

## Components

* Application : 배포할 소스
* Deployment Group : 어떤한 대상에게 배포할 것인지. 타겟 서버들을 하나로 묶은 개념. dev, test, prod staging 전략을 쓸 때 dev deployment 그룹, test deployment 그룹, prod deployment 그룹으로 나눌 수 있음
* Deployment : 배포가 나가는 행위
* Deployment Configuration : 배포가 나갈 때 어떤 형태로 나갈 것인지(Blue/Green? Canary? 등 어떤 형태로 배포될 것인지 설정).

## Continuous Deployment VS Continuous Delivery

* Continuous Deployment는 기술적인 측면이다. 즉 SW가 모두 배포된 것을 말하고 Continuous Delivery는 비지니스 측면이다. Human decision이 개입했냐의 여부로 이 둘의 차이점을 따진다. 어떠한 이벤트를 할 것이라고 했을 때 배포를 하고 나서 관리자가 수동으로 `ok or cancel`을 함으로써 이벤트 시점(배포 시점)을 결정할 수 있다.

## CodeDeploy Agent

CodeDeploy를 통해서 target server에 배포를 하려면 target server에는 CodeDeploy Agent가 설치되어야 한다. CodeDeploy는 appspec.yml에 명세되어 있는 대로 배포를 하라고 **CodeDeploy Agent**에게 명령을 내릴 뿐이다. 그래서 CodeDeploy Agent가 S3에 쌓여있는 데이터를 읽어서 배포를 진행하기 때문에 CodeDeploy Agent가 설치되어 있는 EC2에 S3에 접근할 수 있는 IAM을 부여해야 한다.

## 환경설정

AWS Service와 통신할 때는 IAM 권한이 필요하다. 그래서 Jenkins(EC2)와 AWS CodeBuild에게 필요한 IAM권한을 부여해야 한다. 그리고 이 둘은 S3를 통해서 필요한 소스 코드를 공유한다.

### Jenkins(EC2) IAM

* S3 : S3에 소스 코드를 업로드할 수 있는 권한
* CodeBuild : EC2에서 codeBuild를 실행하는 권한
* CodeDeploy : EC2에서 codeDeploy를 실행하는 권한

### CodeBuild IAM

* S3 : Jenkins가 업로드한 소스 코드에 접근할 수 있는 권한 & CodeDeploy에게 어떠한 파일을 배포하라고 알려주기 위해서 다시 CodeBuild가 파일을 S3에 저장
* AWS ECR : Docker image를 빌드하고 ECR에 배포할 수 있는 권한
* AWS CloudWatch : CodeBuild에 대한 로그는 CloudWatch에 남기 때문에 CloudWatch에 대한 권한 필요
* VPC : CodeBuild는 private subnet에 위치 시킬 것이므로 VPC에 대한 권한 필요

### Target Server

* S3 : CodeDeploy Agent가 업로드한 소스 코드에 접근할 수 있는 권한. 배포 작업을 하는 것은 CodeDeploy가 아니라 Agent다.

**실습파일**

```shell
git clone https://github.com/hongjunjeong95/devops_06_03_jenkins.git
```

## Infra

1. infra/apne2/ec2/jenkins
2. infra/apne2/codebuild/jenkins-slave
3. infra/apne2/codedeploy/jenkins-slave
4. infra/apne2/ec2/target


## Deploy the Image to CodeDeploy(CD)

1. Jenkins plugin **설치 가능**에서 **AWS CodeBuild** plugin 설치

2. Write JenkinsFile

   ```yaml
   pipeline {
     agent { label 'master' }
   
     parameters {
       booleanParam(name : 'BUILD_DOCKER_IMAGE', defaultValue : true, description : 'BUILD_DOCKER_IMAGE')
       booleanParam(name : 'PROMPT_FOR_DEPLOY', defaultValue : false, description : 'PROMPT_FOR_DEPLOY')
       booleanParam(name : 'DEPLOY_WORKLOAD', defaultValue : true, description : 'DEPLOY_WORKLOAD')
   
       // CI
       string(name : 'AWS_ACCOUNT_ID', defaultValue : '264403635125', description : 'AWS_ACCOUNT_ID')
       string(name : 'DOCKER_IMAGE_NAME', defaultValue : 'demo', description : 'DOCKER_IMAGE_NAME')
       string(name : 'DOCKER_TAG', defaultValue : '1.0.0', description : 'DOCKER_TAG')
   
       // CD
       string(name : 'TARGET_SVR_USER', defaultValue : 'ec2-user', description : 'TARGET_SVR_USER')
       string(name : 'TARGET_SVR_PATH', defaultValue : '/home/ec2-user/', description : 'TARGET_SVR_PATH')
       string(name : 'TARGET_SVR', defaultValue : '10.0.3.61', description : 'TARGET_SVR')
     }
   
     environment {
       REGION = "ap-northeast-2"
       ECR_REPOSITORY = "${params.AWS_ACCOUNT_ID}.dkr.ecr.ap-northeast-2.amazonaws.com"
       ECR_DOCKER_IMAGE = "${ECR_REPOSITORY}/${params.DOCKER_IMAGE_NAME}"
       ECR_DOCKER_TAG = "${params.DOCKER_TAG}"
   
       CODEBUILD_NAME = "jenkins-slave-lewisjeong-codebuild"
       CODEBUILD_ARTIFACT_S3_NAME = "jenkins-slave-artifact-codebuild-s3"
       CODEBUILD_ARTIFACT_S3_KEY = "${currentBuild.number}/jenkins-slave-lewisjeong-codebuild"
       CODEDEPLOY_NAME = "demo-codedeploy-app"
       CODEDEPLOY_GROUP_NAME = "dev-codedeploy-group"
     }
   
     stages {
       stage('============ Build Docker Image ============') {
           when { expression { return params.BUILD_DOCKER_IMAGE } }
           agent { label 'master' }
           steps {
               awsCodeBuild(
                 credentialsType: 'keys',
                 region: "${REGION}",
                 projectName: "${CODEBUILD_NAME}",
                 sourceControlType: 'jenkins',
                 sseAlgorithm: 'AES256',
                 buildSpecFile: "deploy/buildspec.yml",
                 artifactTypeOverride: "S3",
                 artifactNamespaceOverride: "NONE",
                 artifactPackagingOverride: "ZIP",
                 artifactPathOverride: "${currentBuild.number}", // jenkins에서 제공하는 변숟(currentBuild.number)
                 artifactLocationOverride: "${CODEBUILD_ARTIFACT_S3_NAME}"
               )
           }
       }
       stage('Prompt for deploy') {
           when { expression { return params.PROMPT_FOR_DEPLOY } }
           agent { label 'master' }
           steps {
               script {
                   env.APPROAL_NUM = input message: 'Please enter the approval number',
                                           parameters: [string(defaultValue: '', description: '', name: 'APPROVAL_NUM')]
               }
   
               echo "${env.APPROAL_NUM}"
           }
       }
       stage('============ Deploy workload ============') {
           when { expression { return params.DEPLOY_WORKLOAD } }
           agent { label 'master' }
           steps {
               echo "Run CodeDeploy with creating deployment"
               script {
                   sh"""
                       aws deploy create-deployment \
                           --application-name ${CODEDEPLOY_NAME} \
                           --deployment-group-name ${CODEDEPLOY_GROUP_NAME} \
                           --region ${REGION} \
                           --s3-location bucket=${CODEBUILD_ARTIFACT_S3_NAME},bundleType=zip,key=${CODEBUILD_ARTIFACT_S3_KEY} \
                           --file-exists-behavior OVERWRITE \
                           --output json > DEPLOYMENT_ID.json
                   """
                   def DEPLOYMENT_ID = sh(script: "cat DEPLOYMENT_ID.json | grep -o '\"deploymentId\": \"[^\"]*' | cut -d'\"' -f4", returnStdout: true).trim()
                   echo "$DEPLOYMENT_ID"
                   sh "rm -rf ./DEPLOYMENT_ID.json"
                   def DEPLOYMENT_RESULT = ""
                   while("$DEPLOYMENT_RESULT" != "\"Succeeded\"") {
                       DEPLOYMENT_RESULT = sh(
                           script:"aws deploy get-deployment \
                                       --region ${REGION} \
                                       --query \"deploymentInfo.status\" \
                                       --deployment-id ${DEPLOYMENT_ID}",
                           returnStdout: true
                       ).trim()
                       echo "$DEPLOYMENT_RESULT"
                       if ("$DEPLOYMENT_RESULT" == "\"Failed\"") {
                           currentBuild.result = 'FAILURE'
                           break
                       }
                       sleep(10) // sleep 10s
                   }
                   currentBuild.result = 'SUCCESS'
               }
           }
       }
     }
   
     post {
       success {
         slackSend(
           channel: "#jenkins_test",
           color: "good",
           message: "[Successful] Job:${env.JOB_NAME}, Build num:#${env.BUILD_NUMBER} (<${env.RUN_DISPLAY_URL}|open job detail>)"
         )
       }
       failure {
         slackSend(
           channel: "#jenkins_test",
           color: "danger",
           message: "[Failed] Job:${env.JOB_NAME}, Build num:#${env.BUILD_NUMBER} @channel (<${env.RUN_DISPLAY_URL}|open job detail>)"
         )
       }
     }
   }
   ```

3. Jenkinsfile에서 중요 부분(Build)

   ```yaml
   stages {
       stage('============ Build Docker Image ============') {
           steps {
           		// Jenkins에서 aws codeBulid에 어떤 프로젝트에 접근하고 action을 취할지 씀
               awsCodeBuild(       
                 artifactTypeOverride: "S3",
                 artifactNamespaceOverride: "NONE",
                 artifactPackagingOverride: "ZIP",
                 artifactPathOverride: "${currentBuild.number}", // jenkins에서 제공하는 변숟(currentBuild.number)
                 artifactLocationOverride: "${CODEBUILD_ARTIFACT_S3_NAME}"
               )
           }
       }
   }
   ```

   * artifact 추가

4. Jenkinsfile에서 중요 부분(Deploy)

   ```yaml
   stage('============ Deploy workload ============') {
           steps {
               echo "Run CodeDeploy with creating deployment"
               script {
                   sh"""
                       aws deploy create-deployment \
                           --application-name ${CODEDEPLOY_NAME} \
                           --deployment-group-name ${CODEDEPLOY_GROUP_NAME} \
                           --region ${REGION} \
                           --s3-location bucket=${CODEBUILD_ARTIFACT_S3_NAME},bundleType=zip,key=${CODEBUILD_ARTIFACT_S3_KEY} \
                           --file-exists-behavior OVERWRITE \
                           --output json > DEPLOYMENT_ID.json
                   """
                   def DEPLOYMENT_ID = sh(script: "cat DEPLOYMENT_ID.json | grep -o '\"deploymentId\": \"[^\"]*' | cut -d'\"' -f4", returnStdout: true).trim()
                   echo "$DEPLOYMENT_ID"
                   sh "rm -rf ./DEPLOYMENT_ID.json"
                   def DEPLOYMENT_RESULT = ""
                   while("$DEPLOYMENT_RESULT" != "\"Succeeded\"") {
                       DEPLOYMENT_RESULT = sh(
                           script:"aws deploy get-deployment \
                                       --region ${REGION} \
                                       --query \"deploymentInfo.status\" \
                                       --deployment-id ${DEPLOYMENT_ID}",
                           returnStdout: true
                       ).trim()
                       echo "$DEPLOYMENT_RESULT"
                       if ("$DEPLOYMENT_RESULT" == "\"Failed\"") {
                           currentBuild.result = 'FAILURE'
                           break
                       }
                       sleep(10) // sleep 10s
                   }
                   currentBuild.result = 'SUCCESS'
               }
           }
       }
   ```

5. deploy/buildspec.yml

   ```yaml
   version: 0.2
   env:
     variables:
       AWS_DEFAULT_REGION: "ap-northeast-2"
       ECR_URL: "264403635125.dkr.ecr.ap-northeast-2.amazonaws.com"
       ECR_DOCKER_IMAGE: "demo"
       ECR_DOCKER_TAG: "1.0.0"
   
   phases:
     pre_build:
       commands:
         - echo Logging in to Amazon ECR...
         - aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${ECR_URL}
     build:
       on-failure: ABORT
       commands:
         - echo Build started on `date`
         - echo Building the Docker image...
         - docker build -t ${ECR_URL}/${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG} .
     post_build:
       commands:
         - echo Build completed on `date`
         - echo Pushing the Docker image...
         - docker push ${ECR_URL}/${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG}
   artifacts: # Agent가 파일들을 사용할 수 있도록 CodeDeploy가 S3(artifacts)에 파일을 업로드
     files:
       - deploy/appspec.yml
       - deploy/scripts/kill_process.sh
       - deploy/scripts/run_process.sh
       - deploy/docker-compose.yml
     discard-paths: yes # 위 deploy/scripts 같은 path들을 다 없애고 root directory에 파일들을 저장
   ```

6. deploy/appspec.yml

   ```yaml
   version: 0.0
   os: linux
   files:
     - source: / # S3에 있는 경로
       destination: /home/ec2-user # 어떤 경로에 copy할지
       overwrite: yes
   permission:
     - object: /home/ec2-user 
       pattern: "**" # 위 오브젝트 디렉토리 안에 있는 전체 파일
       owner: ec2-user
       group: ec2-user
   hooks:
     ApplicationStop:
       - location: kill_process.sh # 앱이 stop할 때 어떤 script를 실행할지
         timeout: 100
         runas: ec2-user
     ApplicationStart:
       - location: run_process.sh # 앱이 start할 때 어떤 script를 실행할지
         timeout: 3600
         runas: ec2-user
   ```

   * CodeDeploy Agent가 어떻게 배포할 것인지에 대한 명세서
   * Ref : https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/reference-appspec-file.html

7. deploy/scripts/kill_process.sh

   ```sh
   #!/bin/bash
   
   echo "Remove existed container"
   docker-compose -f /home/ec2-user/docker-compose.yml down || true
   ```

8. deploy/scripts/run_process.sh

   ```sh
   #!/bin/bash
   
   REGION="ap-northeast-2"
   ACCOUNT_ID="264403635125"
   ECR_REPOSITORY="${ACCOUNT_ID}.dkr.ecr.ap-northeast-2.amazonaws.com"
   ECR_DOCKER_IMAGE="${ECR_REPOSITORY}/demo"
   ECR_DOCKER_TAG="1.0.0"
   
   aws ecr get-login-password --region ${REGION} \
     | docker login --username AWS --password-stdin ${ECR_REPOSITORY};
   
   export IMAGE=${ECR_DOCKER_IMAGE};
   export TAG=${ECR_DOCKER_TAG};
   docker-compose -f /home/ec2-user/docker-compose.yml up -d;
   ```


## CodeDeploy with multiple target server

1. Create multiple target server with terraform

2. codedeploy/jenkins-slave/deployment_config.tf

   ```yaml
   # Deployment를 어떻게 할지에 대한 설정
   
   resource "aws_codedeploy_deployment_config" "this" {
     deployment_config_name = local.deployment_config_name
     compute_platform = local.compute_platform
   
     minimum_healthy_hosts {
       type  = "HOST_COUNT"
       value = 1
     }
   
   #  minimum_healthy_hosts {
   #    type = "FLEET_PERCENT"
   #    value = 50
   #  }
   }
   ```

3. codedeploy/jenkins-slave/deployment_group.tf

   ```yaml
   resource "aws_codedeploy_deployment_group" "this" {
     app_name              = aws_codedeploy_app.this.name
     deployment_group_name = local.deployment_group_name
     service_role_arn      = module.iam.iam_role_arn
     # 여러 개의 target server에 배포할 때 deployment_config_name 주석 해제
     deployment_config_name = aws_codedeploy_deployment_config.this.id # Add
   }
   ```

4. codedeploy/jenkins-slave/_terraform.auto.tfvars에 밑에 코드가 추가

   ```yaml
   ec2_tag_filter = [
     {
       key   = "Name"
       type  = "KEY_AND_VALUE"
       value = "target-ec2"
     },
     {
       key   = "Name"
       type  = "KEY_AND_VALUE"
       value = "target-0-ec2"
     },
     {
       key   = "Name"
       type  = "KEY_AND_VALUE"
       value = "target-1-ec2"
     },
     {
       key   = "Name"
       type  = "KEY_AND_VALUE"
       value = "target-2-ec2"
     }
   ]
   ```

   * deploy를 할 ec2 instance 당 하나씩 추가한다.

5. ec2/templates/userdata.sh

   ```sh
   #!/bin/bash
   
   sudo yum update
   sudo yum install -y docker git java-1.8.0-openjdk ruby wget
   
   cd /home/ec2-user
   wget https://aws-codedeploy-ap-northeast-2.s3.amazonaws.com/latest/install
   chmod +x ./install
   sudo ./install auto
   sudo service codedeploy-agent status
   rm -rf ./install
   
   cat >/etc/init.d/codedeploy-start.sh <<EOL
   #!/bin/bash
   sudo service codedeploy-agent restart
   EOL
   chmod +x /etc/init.d/codedeploy-start.sh
   
   # ec2-user에게도 docker cli를, 즉 docker socket에 접근할 수 잇는 권한이 필요
   sudo usermod -aG docker ec2-user
   systemctl enable docker
   systemctl start docker
   
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   chmod +x /usr/local/bin/docker-compose
   ```
