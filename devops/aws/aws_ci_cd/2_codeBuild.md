# CodeBuild

Jenkins를 이용해서 build와 deploy를 관리하면 build용 jenkins(ec2)와 deploy용 jenkins(ec2)를 관리해야 하는 운영 cost가 발생한다. 그래서 AWS의 codePipeline, codeBuild, codeDeploy를 활용할 것이다.

## 장점

* 빌드를 담당하는 jenkins slave를 관리할 필요가 없어진다.
* CodeBuild 자체 내에서 컨테이너를 띄워서 독립적인 환경에서 빌드를 구축할 수 있음

## 환경설정

AWS Service와 통신할 때는 IAM 권한이 필요하다.

### CodeBuild IAM

* S3 : CodePipe이 업로드한 소스 코드에 접근할 수 있는 권한
* AWS CloudWatch : CodeBuild에 대한 로그는 CloudWatch에 남기 때문에 CloudWatch에 대한 권한 필요
* VPC : CodeBuild는 private subnet에 위치 시킬 것이므로 VPC에 대한 권한 필요

## Create Artifact(S3)

<img src="https://user-images.githubusercontent.com/33750210/145543085-6932c432-058d-4b56-938b-d87aba51112a.png" alt="image" style="zoom:50%;" />

* bucket name 설정
* region 설정

<img src="https://user-images.githubusercontent.com/33750210/145543203-4deeafec-50f9-4b62-b530-de552a67d326.png" alt="image" style="zoom:50%;" />

* ACLs disable
* Block all public access

<img src="https://user-images.githubusercontent.com/33750210/145543301-5ef9a8f2-50fb-4310-a1b6-49bc7100ecae.png" alt="image" style="zoom:50%;" />

* Bucket Versioning : Disable
* Tag
  * Name : demo-artifact-s3
* Default encryption : Disable

**Create bucket**

## Create CodeBuild

<img src="https://user-images.githubusercontent.com/33750210/145486149-203cf829-3576-4233-92a9-581cc9f14ead.png" alt="image" style="zoom:50%;" />

**Build projects** 선택

![image](https://user-images.githubusercontent.com/33750210/145486751-63aa2533-9c71-45b5-b03e-7666e8a8e113.png)

**Create build project** 선택

<img src="https://user-images.githubusercontent.com/33750210/145488134-da6caf7a-deb8-4143-8581-6ebb386ac180.png" alt="image" style="zoom:50%;" />

* Project name : 이름
* Build badge : AWS CodeBuild는 이제 동적으로 생성되어 프로젝트의 최신 빌드 상태를 표시하는 내장 가능형 이미지(*배지*)를 제공하는 빌드 배지 사용을 지원한다. 이 이미지는 CodeBuild 프로젝트를 위해 생성된 URL을 통해 공개적으로 액세스할 수 있다. 이를 통해 누구나 CodeBuild 프로젝트의 상태를 볼 수 있다. 빌드 배지에는 보안 정보가 포함되어 있지 않으므로 인증이 필요하지 않다.
* Enable concurrent build limit : 이 프로젝트에 동시에 빌드할 수 있는 허용 수를 제한. 체크 박스를 on 한면 개수 설정 가능
* Tags : 태그 추가

<img src="https://user-images.githubusercontent.com/33750210/145500177-6a65fca5-0cb7-49a9-a3dc-04db3433eb90.png" alt="image" style="zoom:50%;" />

* Source provider 선택. 여기서는 CodeCommit 선택
  * 종류 : Amazon S3, AWS CodeCommit, GitHub, Bitbucket, GitHub Enterprise
* Reference type : Branch
* Brach : Master or another branch
* Git clone depth : Commit 개수를 얼마나 가져올지 정한다. 이 소스로 개발할 것이라면 Full을 권장한다.
* Git submodules : uncheck

<img src="https://user-images.githubusercontent.com/33750210/145502339-9135ae23-97e3-4727-889a-53170baaf294.png" alt="image" style="zoom:50%;" />

* Managed image
* OS : Ubuntu
* Runtime : Standard
* Image : aws/codebuild/standard:4.0
* Image version : Always use the latest image for this runtime version
* Environment type : Linux
* Privileged : Check

<img src="https://user-images.githubusercontent.com/33750210/145502701-859da18e-4340-46ec-ba81-83bf442a6672.png" alt="image" style="zoom:50%;" />

* Service role : New service role 선택. CodeBuild에 대한 IAM 권한을 부여한다.
* Role name : 이름 설정
* 그 외 설정은 위와 같이 Default로 놔두던가 customizing 하면 된다.

<img src="https://user-images.githubusercontent.com/33750210/145502810-d0f10569-c446-4a71-bbcc-7c290b8f1af2.png" alt="image" style="zoom:45%;" />

VPC 설정이다. Bastion 인스턴스를 제외하고 보통 public subnet에 instance를 두는 경우는 없다. private subnet에 codeBuild를 추가한다.

<img src="https://user-images.githubusercontent.com/33750210/145503068-7b0bfee6-b61e-4a95-83ec-d610fe567b59.png" alt="image" style="zoom:50%;" />

CodeBuild는 **buildspec.yml** 파일을 읽어서 어떻게 code를 build 할지를 수행한다.

<img src="https://user-images.githubusercontent.com/92770273/146720899-727a6881-c630-473f-9530-bda3d1d8a98f.png" alt="image" style="zoom:60%;" />

일괄적으로 동시에 build를 할지 설정한다.

<img src="https://user-images.githubusercontent.com/33750210/145508304-869ba5eb-965f-4342-ad2a-24bcfcfc0f94.png" alt="image" style="zoom:50%;" />

Artifacts는 소스 코드를 공유하는 저장소다. CodeBuild가 build된 소스 코드를 artifact(S3)에 upload하고 codeDeploy agent가 여기에 접근해서 소스 코드를 가져와서 배포 작업을 진행한다.

* Type : Amazon S3
* Bucket name : 미리 생성해둔 버킷. 하지만 CodePipeline에서 artifact를 설정할 것이기 때문에 여기서는 bucket을 지정하지 않는다.
* Name : 파일이나 압축 파일이 생성될 때 저장될 경로
* Enable semantic versioning : `buildspec.yml`에 정의된 artifact name으로 파일 저장
* Path : 빌드 output zip 파일 또는 폴더의 경로다.
* Namespace type : 저장될 zip파일 또는 폴더의 이름 타입이다.
* CodePipeline을 이용할 것이기 때문에 **No artifacts**로 다시 변경

<img src="https://user-images.githubusercontent.com/33750210/145508491-5b15626e-6fec-4f53-bd3c-88f9ff452119.png" alt="image" style="zoom:50%;" />

* CloudWatch로 build 과정을 확인 가능
* **Create build project**

## Push the Image to CodeBuild(CI)

1. deploy/buildspec.yml

   ```yaml
   version: 0.2
   phases:
     pre_build:
       commands:
         - echo Logging in to Ubuntu ECR...
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
   
5. `git push`

6. CloudWatch

   <img src="https://user-images.githubusercontent.com/33750210/145154073-0c7198fb-4bec-4898-ade0-fd30189d0c35.png" alt="image" style="zoom:50%;" />

   1. 이 모든 것이 CloudWatch에 기록
   4. CloudWatch에서 특정 log를 subscription해서 람다를 통해서 이벤트를 쏠 수 있다.
