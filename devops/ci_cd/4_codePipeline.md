# CodePipeline

Jenkins를 이용해서 build와 deploy를 관리하면 build용 jenkins(ec2)와 deploy용 jenkins(ec2)를 관리해야 하는 운영 cost가 발생한다. 그래서 AWS의 codePipeline, codeBuild, codeDeploy를 활용할 것이다.

## 장점

* 빌드를 담당하는 jenkins slave를 관리할 필요가 없어진다. 즉 운영 코스트를 줄일 수 있음

## CodePipelin Stage

1. Source(Github)
2. Build(CodeBuild)
3. Deploy(CodeDeploy)
4. Approval(Continuous Delivery)
5. Invoke(Lambda, Step Function)
6. Test : CodeBuild에서도 가능하지만 별도의 영역에서도 가능

## 환경설정

AWS Service와 통신할 때는 IAM 권한이 필요하다. 그래서 Jenkins(EC2)와 AWS CodeBuild에게 필요한 IAM권한을 부여해야 한다. 그리고 이 둘은 S3를 통해서 필요한 소스 코드를 공유한다.

### CodePipeline IAM

* S3 : S3에 소스 코드를 업로드할 수 있는 권한
* CodeStar : source 대상자에 대한 접근 권한(ex GitHub)
* CodeBuild : EC2에서 codeBuild를 실행하는 권한
* CodeDeploy : EC2에서 codeDeploy를 실행하는 권한

### CodeBuild IAM

* S3 : Jenkins가 업로드한 소스 코드에 접근할 수 있는 권한 & CodeDeploy에게 어떠한 파일을 배포하라고 알려주기 위해서 다시 CodeBuild가 파일을 S3에 저장
* AWS ECR : Docker image를 빌드하고 ECR에 배포할 수 있는 권한
* AWS CloudWatch : CodeBuild에 대한 로그는 CloudWatch에 남기 때문에 CloudWatch에 대한 권한 필요
* VPC : CodeBuild는 private subnet에 위치 시킬 것이므로 VPC에 대한 권한 필요

### CodeDeploy IAM

* S3 : CodeBuild가 업로드한 소스 코드에 접근할 수 있는 권한

### Target Server

* S3 : CodeDeploy Agent가 업로드한 소스 코드에 접근할 수 있는 권한. 배포 작업을 하는 것은 CodeDeploy가 아니라 Agent다.

**실습파일**

```shell
git clone https://github.com/hongjunjeong95/devops_06_03_jenkins.git
```

## Infra

1. infra/apne2/codebuild/jenkins-slave
3. infra/apne2/codedeploy/jenkins-slave
3. infra/apne2/codepipeline/jenkins-slave
4. infra/apne2/ec2/target


## CodePipeline with Notification

1. Create notification rule

   ![image](https://user-images.githubusercontent.com/33750210/145371529-817d31e1-0701-4070-b9dd-48682c3b90b6.png)

2. Notification 설정

   <img src="https://user-images.githubusercontent.com/33750210/145371741-4bd36d42-b236-4a47-848f-a3eb039775ae.png" alt="image" style="zoom:50%;" />

   * Notification name
   * Detail type : Full
   * Events that trigger notifications : 어떤 event가 왔을 때 notification을 받을지 선택

3. Targets 설정

   <img src="https://user-images.githubusercontent.com/33750210/145371957-70860100-d8a1-4556-907e-06c6e5392b80.png" alt="image" style="zoom:50%;" />

   * target이 없다면 **Create target**

   * Create target

     <img src="https://user-images.githubusercontent.com/33750210/145372189-58dd6855-935d-4515-8f14-7f2a8b7a956e.png" alt="image" style="zoom:50%;" />

4. Notification 생성 완료

   <img src="https://user-images.githubusercontent.com/33750210/145372688-c001cf11-dc3f-4ea9-a048-f32413859769.png" alt="image" style="zoom:50%;" />

   * 위 event로 notification이 생성
   * 이제 이 sns 메시지를 받아서 slack에 notification을 쏴줄 chatbot을 생성해야 한다.

5. Chatbot 생성

   <img src="https://user-images.githubusercontent.com/33750210/145372911-517ee4b4-9988-4bcb-b5ed-c1e037103a67.png" alt="image" style="zoom:50%;" />

   * [AWS Chatbot](https://us-east-2.console.aws.amazon.com/chatbot/home#/) 접속
   * Slacke 선택 후 **Configure client**

6. 워크 스페이스 선택

   <img src="https://user-images.githubusercontent.com/33750210/145373225-bdf7fc42-cf0d-4ed9-b831-70eb1a3dc7ae.png" alt="image" style="zoom:40%;" />

   * 우측 상단에서 슬랙 워크페이스 선택
   * 허용

7. Configure new channel

   ![image](https://user-images.githubusercontent.com/33750210/145373379-15406e3f-d720-440f-83f8-db604784220d.png)

### Chatbot Configuration

<img src="https://user-images.githubusercontent.com/33750210/145373811-300554ab-149c-4e01-9900-4edaf44adcbc.png" alt="image" style="zoom:50%;" />

1. Configuration details
   * name : demo-noti
2. Slack channel
   * Channel type : your choice
   * Public(Private) channel name

<img src="https://user-images.githubusercontent.com/33750210/145374104-e6121523-d579-4e6d-be86-9e825b35cb1e.png" alt="image" style="zoom:40%;" />

1. Chnnel IAM role
2. Create an IAM role using a template
3. Role name : DemoChatBotRole
4. Policy templates : Notification permissions

![image](https://user-images.githubusercontent.com/33750210/145374556-f6067bcd-2ecb-4540-936e-ad5000d7cdd2.png)

* AWSCodeStarNotificationsServiceRolePolicy 선택

<img src="https://user-images.githubusercontent.com/33750210/145374313-a208f877-319f-49ed-a564-1e2a3ef83f29.png" alt="image" style="zoom:50%;" />

1. Region : ap-northeast-2
2. Topics : demo-noti-sns(AWS SNS에서 생성한 토픽)

### Chatbot 등록 확인

1. AWS SNS

![image](https://user-images.githubusercontent.com/33750210/145375019-f3c0742e-b085-4af9-9e63-ed42b2db2c09.png)

AWS SNS에 가보면 엔드포인트에 해당 chatbot이 등록되어 있음

2. CodePipeline

![image](https://user-images.githubusercontent.com/33750210/145375203-798ea762-e913-467c-8f69-533622ef10ab.png)

* CodePipeline에서 SNS가 Active로 잘 등록되어 있음
