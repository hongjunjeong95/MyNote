# CodePipeline

## CodePipelin Stage

1. Source(CodeCommit)
2. Build(CodeBuild)
3. Deploy(CodeDeploy)
4. Approval(Continuous Delivery)
5. Invoke(Lambda, Step Function)
6. Test : CodeBuild에서도 가능하지만 별도의 영역에서도 가능

## 환경설정

AWS Service와 통신할 때는 IAM 권한이 필요하다.

### CodePipeline IAM

- S3 : S3에 소스 코드를 업로드할 수 있는 권한
- CodeStar : source 대상자에 대한 접근 권한(ex GitHub)
- CodeBuild : EC2에서 codeBuild를 실행하는 권한
- CodeDeploy : EC2에서 codeDeploy를 실행하는 권한

## Create Codepipeline Role

<img src="https://user-images.githubusercontent.com/33750210/145545861-2469e55b-15f9-4af6-9f96-e384a73dc806.png" alt="image" style="zoom:50%;" />

위 권한을 가진 role 생성

<img src="https://user-images.githubusercontent.com/33750210/145544356-eb45ca69-be92-44c7-aa28-266c8b47d344.png" alt="image" style="zoom:50%;" />

**Edit trust relationship**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "codepipeline.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Trust Relationship을 위 코드로 수정

## Create CodePipeline

![image](https://user-images.githubusercontent.com/33750210/145541926-59794365-352c-4a5c-8828-80f61b34138d.png)

1. 좌측 nav-bar에서 Pipelines 선택
2. **Create pipeline** 선택

<img src="https://user-images.githubusercontent.com/33750210/145544822-097b3056-0a8c-4c5f-bc3a-12eb674c5afa.png" alt="image" style="zoom:50%;" />

1. 이름 설정
2. Service role : 위에서 만든 role 선택
3. Artifact store : 위에서 만든 S3 bucket 선택
4. Encryption key : Default AWS Managed Key 선택
5. Next

<img src="https://user-images.githubusercontent.com/33750210/145545114-73b9bfc5-a0af-4f9d-ba0b-1d3ddd9926fa.png" alt="image" style="zoom:50%;" />

1. Source provider : AWS CodeCommit
2. Repository name : demo-commit(CodeCommit에 존재하는 repository)
3. Branch name : master
4. Next

<img src="https://user-images.githubusercontent.com/33750210/145545346-8ec6d24a-8d80-408d-a646-d2e011b12fba.png" alt="image" style="zoom:50%;" />

1. AWS provider : AWS CodeBuild
2. Region 선택
3. Project name : CodeBuild project 선택
4. Build type : Single build
5. Next

<img src="https://user-images.githubusercontent.com/33750210/145545509-6759eb45-bde6-41b0-814e-7e510ef5bf35.png" alt="image" style="zoom:50%;" />

1. Deploy provider : AWS CodeDeploy
2. Region : Seoul
3. Application name : your codeDeploy app name
4. Deployment group : your codeDeploy group name
5. Next
