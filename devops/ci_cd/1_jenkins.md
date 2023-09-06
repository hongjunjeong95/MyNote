# Jenkins



**실습파일**

```shell
git clone https://github.com/hongjunjeong95/devops_06_03_jenkins.git
```

## Infra

1. infra/apne2/network/vpc

2. infra/apne2/ec2/bastion

3. infra/apne2/ec2/jenkins

4. infra/apne2/alb/jenkins

```shell
# Execute the commands above the diretories
tf init
tf apply --auto-approve
```

## Install Jenkins

1. Edit `~/.ssh/config`

   ```yaml
   Host bastion
     HostName <bastion_public_ip>
     User ec2-user
     IdentityFile <key_pair_path>
   ```

2. jenkins 설치

   ```shell
   cat ~/.ssh/dev.pem | pbcopy
   ssh bastion
   vim ~/.ssh/dev.pem # jenkins의 key_pair 붙이기
   ssh -i ~/.ssh/dev.pem ec2-user@<jenkins_private_ip>
   git clone https://github.com/hongjunjeong95/devops_06_03_jenkins.git
   cd devops_06_03_jenkins/src/jenkins_remote_package
   sh install.sh
   
   # 밑에 3 가지 명령어로 jenkins가 구동되는지 확인 가능
   journalctl -u jenkins.service -f
   netstat -nlp | grep 8080
   curl -v http://localhost:8080
   
   # docker 구동 학인
   logout
   ssh -i ~/.ssh/dev.pem ec2-user@<jenkins_private_ip>
   docker ps -a
   ```

3. Host에 있는 docker daemon에 jenkins에 있는 docker client를 연결하기

   ```shell
   # docker로 jenkins를 구동하기 위해서 패키지로 설치한 jenkins를 중지
   sudo service jenkins stop
   cd devops_06_03_jenkins/src/jenkins_remote_docker
   make run
   ```

## Configure Jenkins

```shell
cd /home/ec2-user/devops_06_03_jenkins/src/jenkins_remote_docker
make run
```

![image](https://user-images.githubusercontent.com/33750210/144950099-d6a11fd4-9823-4584-bc75-c7249d0d45e1.png)

docker log로 알게 된 위 패스워드를 **복사**



<img src="https://user-images.githubusercontent.com/33750210/144950189-fb260644-3f99-4647-9fc5-c4bbfdb0094d.png" alt="image" style="zoom:40%;" />

붙여넣기 & **Continue**

<img src="https://user-images.githubusercontent.com/33750210/144950290-08cfa6c6-8192-4d6e-9106-8e39ba89c4f4.png" alt="image" style="zoom:33%;" />

**Select plugins to install**을 선택



<img src="https://user-images.githubusercontent.com/33750210/144950394-13d73380-612a-457f-b6aa-f758f82dd3f4.png" alt="image" style="zoom:50%;" />

**Notifications and Publishing**에서 모든 체크 박스 해제



<img src="https://user-images.githubusercontent.com/33750210/144950489-2a9621c9-e17a-4fbe-9117-d43ae0f9cd45.png" alt="image" style="zoom:50%;" />

**User Management and Security**에서 모든 체크 박스 해제



<img src="https://user-images.githubusercontent.com/33750210/144950605-23e81460-18f1-4582-9c30-6c861c70e56a.png" alt="image" style="zoom:50%;" />

Docker image로 빌드했기 때문에 **Build Tools**에서 모든 체크 박스 해제

**Install** 버튼 클릭



<img src="https://user-images.githubusercontent.com/33750210/144951096-b57f4ca6-80d9-4f0d-85cd-7f7a56e00184.png" alt="image" style="zoom:40%;" />

1. 계정 입력
2. **Save and Continue**



<img src="https://user-images.githubusercontent.com/33750210/144951178-aec31ec6-a0e5-4faf-aab8-c40c6ff48f5a.png" alt="image" style="zoom:40%;" />

**Save and Finish**



## Synchronize Github

![image](https://user-images.githubusercontent.com/33750210/144951325-d5267d12-6582-4a37-8102-26daae6fd04e.png)

**새로운 Item** 클릭



<img src="https://user-images.githubusercontent.com/33750210/144951480-f6958a47-9673-46b7-a4c0-122ed231db0e.png" alt="image" style="zoom:50%;" />

1. Select **Pipeline**
2. Enter and item name
3. **Ok**



![image](https://user-images.githubusercontent.com/33750210/144952176-13d7277a-56d1-4c74-a65f-f471f7ab1280.png)

1. Check **GitHub project**
2. Add your project repository

<img src="https://user-images.githubusercontent.com/33750210/144963393-5adaba40-74e3-4f2d-8933-057eab3802af.png" alt="image" style="zoom:40%;" />

**Github hook trigger for GITScm polling**으로 github push 트리거 설정



![image](https://user-images.githubusercontent.com/33750210/144952422-2019fe4c-6e7a-4f33-a180-5ac7f25f18d3.png)

1. Definition : **Pipeline script from SCM**
2. SCM :  **Git**
3. Repository는 위에 설정한 것과 똑같이
4. Credentials을 줘야 jenkins가 github repository에 접근 가능



### Credentials

<img src="https://user-images.githubusercontent.com/33750210/144952857-5ae59a3b-56e6-4682-8369-c4050eb2b0d6.png" alt="image" style="zoom:50%;" />

github에서 repo 부분을 모두 체크하고 token을 생성



<img src="https://user-images.githubusercontent.com/33750210/144953039-383beddc-62de-488c-b3ac-cb14c633d200.png" alt="image" style="zoom:50%;" />

Password에 위에서 생성한 github token을 추가하 나머지 정보 입력 후 **Add**

![image](https://user-images.githubusercontent.com/33750210/144953249-f33fd28e-01bc-464f-b7f0-2a5f81f664c3.png)

다시 **Credentials**를 다음과 같이 세팅



<img src="https://user-images.githubusercontent.com/33750210/144953573-9f8e40b4-e9cd-4b6b-82b4-237c58d542c4.png" alt="image" style="zoom:40%;" />

1. Set **Branch Specifier**
2. Script Path : Jenkins 파일이 있는 위치
3. Uncheck **Lightweight checkout**
4. **저장**

### Register Webhooks

![image](https://user-images.githubusercontent.com/33750210/144954711-5650c02a-e88f-44aa-b0ba-e5c82b7c687c.png)

1. Settings on your repository
2. Webhooks
3. Add webhook

<img src="https://user-images.githubusercontent.com/33750210/144955121-8cd693b0-e6b5-4720-b539-36af10cb49ce.png" alt="image" style="zoom:50%;" />

1. Payload URL : jenkins server address + /github-webhook/ => 마지막에 (/)를 꼭 붙여줘야 한다.
2. Content type : application/json
3. Event trigger : Just the push event
4. **Add webhook**

## Push Image to ECR(CI)

1. Create ECR repository

2. Write JenkinsFile

   ```yaml
   pipeline {
       agent any
   
       parameters {
           booleanParam(name : 'BUILD_DOCKER_IMAGE', defaultValue : true, description : 'BUILD_DOCKER_IMAGE')
           booleanParam(name : 'RUN_TEST', defaultValue : true, description : 'RUN_TEST')
           booleanParam(name : 'PUSH_DOCKER_IMAGE', defaultValue : true, description : 'PUSH_DOCKER_IMAGE')
           booleanParam(name : 'DEPLOY_WORKLOAD', defaultValue : true, description : 'DEPLOY_WORKLOAD')
   
           // CI
           string(name : 'AWS_ACCOUNT_ID', defaultValue : '264403635125', description : 'AWS_ACCOUNT_ID')
           string(name : 'DOCKER_IMAGE_NAME', defaultValue : 'demo', description : 'DOCKER_IMAGE_NAME')
           string(name : 'DOCKER_TAG', defaultValue : '1.0.0', description : 'DOCKER_TAG')
       }
   
       environment {
           REGION = "ap-northeast-2"
           ECR_REPOSITORY = "${params.AWS_ACCOUNT_ID}.dkr.ecr.ap-northeast-2.amazonaws.com"
           ECR_DOCKER_IMAGE = "${ECR_REPOSITORY}/${params.DOCKER_IMAGE_NAME}"
           ECR_DOCKER_TAG = "${params.DOCKER_TAG}"
       }
   
       stages {
           stage('============ Build Docker Image ============') {
               when { expression { return params.BUILD_DOCKER_IMAGE } }
               steps {
                   dir("${env.WORKSPACE}") {
                       sh 'docker build -t ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG} .'
                   }
               }
               post {
                   always {
                       echo "Docker Build Success"
                   }
               }
           }
           stage('============ Run test code ============') {
               when { expression { return params.RUN_TEST } }
               // agent { label 'build' }
               steps {
                   sh'''
                       aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}
                       docker run --rm ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG} /root/.local/bin/pytest -v
                   '''
               }
           }
           stage('============ Push Docker Image ============') {
               when { expression { return params.PUSH_DOCKER_IMAGE } }
               steps {
                   echo "Push Docker Image to ECR!"
                   sh'''
                       aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}
                       docker push ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG}
                   '''
               }
           }
           stage('============ Deploy workload ============') {
               when { expression { return params.DEPLOY_WORKLOAD } }
               steps {
               	echo "Deploy workload success!"
               }
           }
       }
   }
   ```

   

   ## Push Image to ECR(CD)

   1. Jenkins plugin **설치 가능**에서 **ssh-agent** plugin 설치

      <img src="https://user-images.githubusercontent.com/33750210/144977620-7998da5e-971c-4fe8-9f47-d0af39df59c2.png" alt="image" style="zoom:50%;" />

   2. Jenkins 관리 => Manage Credentials

      ![image](https://user-images.githubusercontent.com/33750210/144977896-60b83658-9dba-4235-9c9b-981791e89833.png)

      

   3. Jenkins 클릭

      <img src="https://user-images.githubusercontent.com/33750210/144978086-4b0d7c73-e2fb-416e-987c-440f31788852.png" alt="image" style="zoom:50%;" />

   4. Global credentials 클릭

      <img src="https://user-images.githubusercontent.com/33750210/144978251-d55548a8-cdc1-4759-8644-0aff930d6b5e.png" alt="image" style="zoom:50%;" />

   5. Add Credentials 클릭

      <img src="https://user-images.githubusercontent.com/33750210/144978312-a0350d3c-96af-4835-b876-5f32e7a03aa3.png" alt="image" style="zoom:50%;" />

   6. Credentials key 생성

      ![image](https://user-images.githubusercontent.com/33750210/144978631-8c9581e4-6cb6-4005-ae9a-1db6ea6a6673.png)

      1. ID 생성
      2. ec2에 맞는 username
      3. Private key : 여기서 pem 키를 추가

   7. Write JenkinsFile

      ```yaml
      pipeline {
          agent any
      
          parameters {
              booleanParam(name : 'BUILD_DOCKER_IMAGE', defaultValue : true, description : 'BUILD_DOCKER_IMAGE')
              booleanParam(name : 'RUN_TEST', defaultValue : true, description : 'RUN_TEST')
              booleanParam(name : 'PUSH_DOCKER_IMAGE', defaultValue : true, description : 'PUSH_DOCKER_IMAGE')
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
          }
      
          stages {
              stage('============ Build Docker Image ============') {
                  when { expression { return params.BUILD_DOCKER_IMAGE } }
                  steps {
                      dir("${env.WORKSPACE}") {
                          sh 'docker build -t ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG} .'
                      }
                  }
                  post {
                      always {
                          echo "Docker Build Success"
                      }
                  }
              }
              stage('============ Run test code ============') {
                  when { expression { return params.RUN_TEST } }
                  steps {
                      sh"""
                          aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}
                          docker run --rm ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG} /root/.local/bin/pytest -v
                      """
                  }
              }
              stage('============ Push Docker Image ============') {
                  when { expression { return params.PUSH_DOCKER_IMAGE } }
                  steps {
                      echo "Push Docker Image to ECR!"
                      sh"""
                          aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}
                          docker push ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG}
                      """
                  }
              }
              stage('Prompt for deploy') {
                  steps {
                      // input 'Deploy this?'
      
                      script {
                          env.APPROAL_NUM = input message: 'Please enter the approval number',
                                              parameters: [string(defaultValue: '',
                                                          description: '',
                                                          name: 'APPROVAL_NUM')]
                      }
      
                      echo "${env.APPROAL_NUM}"
                  }
              }
      
              stage('============ Deploy workload ============') {
                  when { expression { return params.DEPLOY_WORKLOAD } }
                  steps {
                  		// Credentials에서 설정한 credentials 키 ID를 credentials:[]에 추가
                  		// scp로 파일을 copy
                  		// ssh로 docker-compose 실행
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
      }
      ```

   


## Notification(Slack)

1. Slack => App => Add Jenkins CI

2. Select a channel

3. Jenkins CI 통합 앱 추가

4. Jenkins App에서 Jenkins 관리 => plugin 관리 => 설치 가능 => slack notification plugin 설치

5. Jenkins 관리 => 시스템 설정 => Slack

   <img src="https://user-images.githubusercontent.com/33750210/144989807-88c50fe2-dbfd-4588-8b6c-bf1746c301ad.png" alt="image" style="zoom:40%;" />

   

   * Workspace : slack의 workspace 이름
   * Credential : Jenkins CI App을 생성할 때 제공하는 token

   

   <img src="https://user-images.githubusercontent.com/33750210/144989523-f6cdfdc6-3bce-4887-9ee7-45f32b238cc9.png" alt="image" style="zoom:45%;" />

   * Default channel : channel 이름
   * Test Connection을 실행해서 **Success** 뜨면 됨
   * 저장

6. Write Jenkinsfile

   ```shell
   pipeline {
       agent any
   
       parameters {
           booleanParam(name : 'BUILD_DOCKER_IMAGE', defaultValue : true, description : 'BUILD_DOCKER_IMAGE')
           booleanParam(name : 'RUN_TEST', defaultValue : true, description : 'RUN_TEST')
           booleanParam(name : 'PUSH_DOCKER_IMAGE', defaultValue : true, description : 'PUSH_DOCKER_IMAGE')
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
       }
   
       stages {
           stage('============ Build Docker Image ============') {
               when { expression { return params.BUILD_DOCKER_IMAGE } }
               steps {
                   dir("${env.WORKSPACE}") {
                       sh 'docker build -t ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG} .'
                   }
               }
               post {
                   always {
                       echo "Docker Build Success"
                   }
               }
           }
           stage('============ Run test code ============') {
               when { expression { return params.RUN_TEST } }
               steps {
                   sh"""
                       aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}
                       docker run --rm ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG} /root/.local/bin/pytest -v
                   """
               }
           }
           stage('============ Push Docker Image ============') {
               when { expression { return params.PUSH_DOCKER_IMAGE } }
               steps {
                   echo "Push Docker Image to ECR!"
                   sh"""
                       aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}
                       docker push ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG}
                   """
               }
           }
           stage('Prompt for deploy') {
               when { expression { return params.PROMPT_FOR_DEPLOY } }
               steps {
                   // input 'Deploy this?'
   
                   script {
                       env.APPROAL_NUM = input message: 'Please enter the approval number',
                                           parameters: [string(defaultValue: '',
                                                       description: '',
                                                       name: 'APPROVAL_NUM')]
                   }
   
                   echo "${env.APPROAL_NUM}"
               }
           }
   
           stage('============ Deploy workload ============') {
               when { expression { return params.DEPLOY_WORKLOAD } }
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
   	
   		# Added this code
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



## Jenkins Master-Slave

1. bastion server에 `~/.ssh/config`에 slave_0, slave_1 등록

2. 권한 부여

   ```shell
   chmod 600 ~/.ssh/config
   ```

   

2. slave_0, slave_1 접속

   ```shell
   sudo su
   yum install docker git java-1.8.0-openjdk -y
   systemctl enable docker
   systemctl start docker
   usermod -aG docker ec2-user
   ```

   slave로 등록되기 위해서는 master에서 ssh로 설정할 때 java가 설치되어 있어야 한다.

3. Jenkins 관리 => 노드 관리 => 신규 노드

4. 신규 노드 생성

   <img src="https://user-images.githubusercontent.com/33750210/144992927-ee793b57-bc06-48f3-94d2-2377a06fa982.png" alt="image" style="zoom:50%;" />

   1. 노드명
   2. Check Permanent Agent
   3. Ok

5. 설정

   <img src="https://user-images.githubusercontent.com/33750210/145129870-9a86e9f8-7bdd-4449-89c1-e95d24bad118.png" alt="image" style="zoom:50%;" />

   * slave의 job executor 개수
   * Remote root directory
   * Lables : slave_0은 build, slave_1은 deploy
   * Usage

   ![image](https://user-images.githubusercontent.com/33750210/145128711-c833ea3a-6522-42ac-a4ec-875368e63e80.png)

   * Launch method : Launch agents via SSH
   * Host : slave ip
   * Credentials : ec2-user(Username 아니여도 됨)
   * Host Key Verification Strategy : Manually trusted key Verification Strategy
   * Save

6. Relaunch agent

   <img src="https://user-images.githubusercontent.com/33750210/145129034-cf18d979-ed44-49b9-99d8-bc445101dfcc.png" alt="image" style="zoom:50%;" />

7. 위와 같은 방법으로 slave1도 생성

8. Master 노드에 라벨 추가

   <img src="https://user-images.githubusercontent.com/33750210/145129215-31cc0955-a36e-439c-889e-b070fa9b37c9.png" alt="image" style="zoom:50%;" />

9. Write Jenkinsfile

   ```yaml
   pipeline {
       // agent 별로 job을 할당
       agent { label 'master '}
   
       parameters {
           booleanParam(name : 'BUILD_DOCKER_IMAGE', defaultValue : true, description : 'BUILD_DOCKER_IMAGE')
           booleanParam(name : 'RUN_TEST', defaultValue : true, description : 'RUN_TEST')
           booleanParam(name : 'PUSH_DOCKER_IMAGE', defaultValue : true, description : 'PUSH_DOCKER_IMAGE')
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
       }
   
       stages {
           stage('============ Build Docker Image ============') {
               when { expression { return params.BUILD_DOCKER_IMAGE } }
               agent { label 'build' }
               steps {
                   dir("${env.WORKSPACE}") {
                       sh 'docker build -t ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG} .'
                   }
               }
               post {
                   always {
                       echo "Docker Build Success"
                   }
               }
           }
           stage('============ Run test code ============') {
               when { expression { return params.RUN_TEST } }
               agent { label 'build' }
               steps {
                   sh"""
                       aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}
                       docker run --rm ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG} /root/.local/bin/pytest -v
                   """
               }
           }
           stage('============ Push Docker Image ============') {
               when { expression { return params.PUSH_DOCKER_IMAGE } }
               agent { label 'build' }
               steps {
                   echo "Push Docker Image to ECR!"
                   sh"""
                       aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}
                       docker push ${ECR_DOCKER_IMAGE}:${ECR_DOCKER_TAG}
                   """
               }
           }
           stage('Prompt for deploy') {
               when { expression { return params.PROMPT_FOR_DEPLOY } }
               agent { label 'deploy' }
               steps {
                   // input 'Deploy this?'
   
                   script {
                       env.APPROAL_NUM = input message: 'Please enter the approval number',
                                           parameters: [string(defaultValue: '',
                                                       description: '',
                                                       name: 'APPROVAL_NUM')]
                   }
   
                   echo "${env.APPROAL_NUM}"
               }
           }
   
           stage('============ Deploy workload ============') {
               when { expression { return params.DEPLOY_WORKLOAD } }
               // agent 추가
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

