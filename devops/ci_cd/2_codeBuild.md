# CodeBuild

Jenkins를 이용해서 build와 deploy를 관리하면 build용 jenkins(ec2)와 deploy용 jenkins(ec2)를 관리해야 하는 운영 cost가 발생한다. 그래서 AWS의 codePipeline, codeBuild, codeDeploy를 활용할 것이다.

## 장점

* 빌드를 담당하는 jenkins slave를 관리할 필요가 없어진다.
* CodeBuild 자체 내에서 컨테이너를 띄워서 독립적인 환경에서 빌드를 구축할 수 있음

## 환경설정

AWS Service와 통신할 때는 IAM 권한이 필요하다. 그래서 Jenkins(EC2)와 AWS CodeBuild에게 필요한 IAM권한을 부여해야 한다. 그리고 이 둘은 S3를 통해서 필요한 소스 코드를 공유한다.

### Jenkins(EC2) IAM

* CodeBuild : EC2에서 codeBuild를 실행하는 권한
* S3 : S3에 소스 코드를 업로드할 수 있는 권한

### CodeBuild IAM

* S3 : Jenkins가 업로드한 소스 코드에 접근할 수 있는 권한
* AWS ECR : Docker image를 빌드하고 ECR에 배포할 수 있는 권한
* AWS CloudWatch : CodeBuild에 대한 로그는 CloudWatch에 남기 때문에 CloudWatch에 대한 권한 필요
* VPC : CodeBuild는 private subnet에 위치 시킬 것이므로 VPC에 대한 권한 필요

**실습파일**

```shell
git clone https://github.com/hongjunjeong95/devops_06_03_jenkins.git
```

## Infra

1. infra/apne2/codebuild/jenkins-slave


## Push the Image to CodeBuild(CI)

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
                 buildSpecFile: "deploy/buildspec.yml"
               )
           }
       }
       stage('Prompt for deploy') {
           when { expression { return params.PROMPT_FOR_DEPLOY } }
           agent { label 'deploy' }
           steps {
   //             input 'Deploy this?'
               script {
                   env.APPROVAL_NUM = input message: 'Please enter the approval number',
                                     parameters: [string(defaultValue: '',
                                                  description: '',
                                                  name: 'APPROVAL_NUM')]
               }
   
               echo "${env.APPROVAL_NUM}"
           }
       }
       stage('============ Deploy workload ============') {
           when { expression { return params.DEPLOY_WORKLOAD } }
           agent { label 'deploy' }
           steps {
               sshagent (credentials: ['aws_ec2_user_ssh']) {
                   sh """#!/bin/bash
                       scp -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no \
                           deploy/docker-compose.yml \
                           ${params.TARGET_SVR_USER}@${params.TARGET_SVR}:${params.TARGET_SVR_PATH};
   
                       ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no \
                           ${params.TARGET_SVR_USER}@${params.TARGET_SVR} \
                           'aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}; \
                            export IMAGE=${ECR_DOCKER_IMAGE}; \
                            export TAG=${ECR_DOCKER_TAG}; \
                            docker-compose -f docker-compose.yml down;
                            docker-compose -f docker-compose.yml up -d';
                   """
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

3. Jenkinsfile에서 중요 부분

   ```yaml
   stages {
       stage('============ Build Docker Image ============') {
           when { expression { return params.BUILD_DOCKER_IMAGE } }
           agent { label 'master' }
           steps {
           		// Jenkins에서 aws codeBulid에 어떤 프로젝트에 접근하고 action을 취할지 씀
               awsCodeBuild(
                 credentialsType: 'keys',
                 region: "${REGION}",
                 projectName: "${CODEBUILD_NAME}",
                 sourceControlType: 'jenkins',
                 sseAlgorithm: 'AES256',
                 buildSpecFile: "deploy/buildspec.yml"
               )
           }
       }
   }
   ```

   

4. deploy/buildspec.yml

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
   artifacts:
     files:
       - deploy/appspec.yml
       - deploy/scripts/kill_process.sh
       - deploy/scripts/run_process.sh
       - deploy/docker-compose.yml
     discard-paths: yes
   
   ```

   * AWS CodeBuild에 대한 명세서
   * pre_build 전 단계로 install 관련 명세서도 있지만 여기서는 필요 없어서 따로 언급하지 않음.

5. git push

6. CloudWatch

   <img src="https://user-images.githubusercontent.com/33750210/145154073-0c7198fb-4bec-4898-ade0-fd30189d0c35.png" alt="image" style="zoom:50%;" />

   1. Jenkins에 코드를 배포
   2. Jenkins가 CodeBuild에게 Build하라고 job을 토스
   3. 이 모든 것이 CloudWatch에 기록
   4. CloudWatch에서 특정 log를 subscription해서 람다를 통해서 이벤트를 쏠 수 있다.
   5. 마지막으로 jenkins slave(deploy)에서 code 를 target server에 deploy
