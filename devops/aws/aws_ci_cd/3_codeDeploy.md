# CodeDeploy

## 장점

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

AWS Service와 통신할 때는 IAM 권한이 필요하다.

### CodeBuild IAM

* S3 : CodeDeploy에게 어떠한 파일을 배포하라고 알려주기 위해서 다시 CodeBuild가 파일을 S3에 저장
* AWS CloudWatch : CodeBuild에 대한 로그는 CloudWatch에 남기 때문에 CloudWatch에 대한 권한 필요
* VPC : CodeBuild는 private subnet에 위치 시킬 것이므로 VPC에 대한 권한 필요
* AWS ECR(ECR 쓸 경우) : Docker image를 빌드하고 ECR에 배포할 수 있는 권한

### Target Server

* S3 : CodeDeploy Agent가 업로드한 소스 코드에 접근할 수 있는 권한. 배포 작업을 하는 것은 CodeDeploy가 아니라 Agent다.

## Create Role(EC2)

<img src="https://user-images.githubusercontent.com/33750210/145532714-fb735c55-1ceb-452f-947c-dedfca36c26b.png" alt="image" style="zoom:40%;" />

EC2의 codeDeploy Agent가 S3 artifact에 접근할 수 있도록 **AmazonS3FullAccess** Role 생성

## Create Role(CodeDeploy)

<img src="https://user-images.githubusercontent.com/33750210/145526067-b9d9b51b-689a-4b42-925d-670926e6a8cc.png" alt="image" style="zoom:40%;" />

위와 같이 **AWSCodeDeployRole** 권한을 가진 Role 생성

## Create Target Server

![image](https://user-images.githubusercontent.com/33750210/145532931-f35c83ae-a3d8-4c4e-8872-ff58b9ee459e.png)

Amazon Linux2로 이미지를 선택하고 User data에 밑에 script를 추가

```sh
#!/bin/bash
sudo yum update
sudo yum install -y git java-1.8.0-openjdk ruby wget
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
```

![image](https://user-images.githubusercontent.com/33750210/145533700-788f0237-3778-452a-9785-1ce5b0995d6c.png)

SSH와 HTTP 포트 허용

![image](https://user-images.githubusercontent.com/33750210/145534488-b50e432f-65f2-4002-9dd6-9e84ebd10c66.png)

<img src="https://user-images.githubusercontent.com/33750210/145534322-8e8221a3-c6e5-46af-a45e-8ab3a4a8f0a3.png" alt="image" style="zoom:50%;" />

EC2가 S3에 접근할 수 있는 role 부착

이와 똑같이 instance 한 개 더 생성

## Create Deployments Configuration

<img src="https://user-images.githubusercontent.com/33750210/145536093-f3e3e9b9-c16f-4f77-b903-9f2e52692590.png" alt="image" style="zoom:50%;" />

**Deployment configurations** 선택

![image](https://user-images.githubusercontent.com/33750210/145536224-298853e2-cc22-47aa-88b5-1526182cd4fe.png)

**Create deployment configuration** 클릭

<img src="https://user-images.githubusercontent.com/33750210/145536328-bc7c3fdd-367c-4677-96e6-1a083fdee8ee.png" alt="image" style="zoom:50%;" />

* 이름 설정
* Compute platform : EC2
* Minimum healty hosts : Number
* Value :1
  * 최소 healty hosts는 최소 value(1)가 동작해야지 배포한다는 뜻이다.

## Create CodeDeploy-app

<img src="/Users/hongjun/Library/Application Support/typora-user-images/image-20211210115842241.png" alt="image-20211210115842241" style="zoom:50%;" />

**Applications** 선택

![image](https://user-images.githubusercontent.com/33750210/145509822-f7a582a4-079e-4058-866a-cbf7c959dc4a.png)

**Create application** 선택

<img src="https://user-images.githubusercontent.com/33750210/145510102-5965d767-15a5-45d5-9ef4-61947cca8025.png" alt="image" style="zoom:50%;" />

1. Application name : app name 설정
2. Compute platform : Server(EC2/On-premises) 선택
   * 종류 : ECS, Lambda, EC2
3. Create application

## Create CodeDeploy-group

![image](https://user-images.githubusercontent.com/33750210/145524926-b3ae4dbd-6351-4c86-9bdf-4e54e29cbb6a.png)

**Create deployment group** 선택

<img src="https://user-images.githubusercontent.com/33750210/145537182-9f8faa51-cdd3-4c13-968e-7dd6eed5af01.png" alt="image" style="zoom:50%;" />

* Deployment group name 설정
* Service role : 미리 생성해 둔 CodeDeploy role 부여

<img src="https://user-images.githubusercontent.com/33750210/145537328-237e4e60-07e8-4804-9e49-64b86b2f68cd.png" alt="image" style="zoom:50%;" />

* In-place와 Blue / green : In-place 선택
  * In-place : 배포할 때 기존의 서버 환경을 off 시키고 새로운 환경으로 업데이트한 후 다시 서버를 on 한다.
  * Blue/green : 새로운 서버 환경이 구축되는 동시에 기존의 서버 환경을 deregister하고 새로운 환경을 register 한다.
* Amazon EC2 instances 선택
* Key-Value : 어떤 instance에 배포할 것인지 filtering 한다.

<img src="https://user-images.githubusercontent.com/33750210/145538217-db9012fe-6892-4a00-b3ce-04bab3b01730.png" alt="image" style="zoom:50%;" />

CodeDeploy는 ec2에 있는 codedeploy agent에게 명령을 내릴 뿐이다. 실제로 배포 작업을 하는 것은 appspec.yml에 명세되어 있는 대로 배포를 하는 agent다.

<img src="https://user-images.githubusercontent.com/33750210/145538376-b2540895-d13d-4f33-947c-db8c016221d6.png" alt="image" style="zoom:50%;" />

1. 배포 환경에서 만든 `demo-deploy-config`를 추가
2. 로드 밸런서 해제
3. **Create deployment group**

## appspec.yml 작성

배포할 때 필요한 파일이 appspec.yml 파일이다. 배포를 할 때 소스코드와 같이 appspec.yml 파일을 묶어서 배포를 하고 ec2 agent가 s3에 배포된 소스 코드를 가져와서 압축을 해제 후 appspec.yml에 명시되어 있는대로 배포 작업을 진행한다.

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

1. deploy/scripts/kill_process.sh

   ```sh
   #!/bin/bash
   
   echo "Remove existed container"
   ```
   
   보통 여기에 docker-compose down 명령어로 도커를 중지시킨다.
   
8. deploy/scripts/run_process.sh

   ```sh
   #!/bin/bash
   
   echo "Run container"
   ```
   
   여기에는 docker-compose up으로 다시 서버를 실행시킨다.
