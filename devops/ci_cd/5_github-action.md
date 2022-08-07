# Github Action

## 구성

* Workflows
  * 자동화(build, test, package, relaease, deploy, ...)
  * 하나 이상의 잡들로 이루어져 있다.
  * 이벤트에 의해서 스케줄링 되거나 트리거 될 수 있다.
* Events
  * 워크플로우를 실행하는 특정 행위다.
  * ex) push, PR or Issue created
  * https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows
* Jobs
  * 같은 runner 위에서 실행되는 step들의 집합이다.
  * 디폴트 세팅으로는 여러 개의 잡은 parallel하게 동작한다.
* Steps
  * 개별적인 테스크
  * `action` or `shell command`가 될 수 있다.
  * 각각이 데이터를 공유한다.
* Actions
  * 가장 작은 portable한 워크플로우다. 워크 플로우도 다른 워크 플로우에서 action이라는 이름으로 호출할 수 있다.
  * standalone commands
* Runners
  * server
  * Github hosted or Self hosted
  * https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions#jobsjob_idruns-on

## 특징

* yaml syntax를 사용해서 events, jobs와 steps를 정의한다.
* .github/workflows가 프로젝트 경로다.
  * .github/workflows/{name}.yml

## 실습

### AWS 키 등록

<img src="https://user-images.githubusercontent.com/33750210/146733464-99e8f36b-ce55-4175-9c5b-68988eebc2d4.png" alt="image" style="zoom:40%;" />

**Settings**

![image](https://user-images.githubusercontent.com/33750210/146733573-9529010a-6892-442a-85d1-7daefc963b0e.png)

**Secretes** => **New repository secret**

<img src="https://user-images.githubusercontent.com/33750210/146733886-9558daa1-be41-41d6-a1a1-3063e183cf52.png" alt="image" style="zoom:50%;" />

### YAML 작성

```yaml
name: CI/CD

on:
  push:
    branches: [master]
    paths-ignore:
      - '.gitignore'
      - '.dockerignore'
      - 'README.md'
    # paths:
  #      - '**.py'
  pull_request:
    branches: [master]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 1

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Set var
        id: set-var
        run: |
          echo ::set-output name=ECR_REGISTRY::${{ steps.login-ecr.outputs.registry }}
          echo ::set-output name=ECR_REPOSITORY::demo
          echo ::set-output name=IMAGE_TAG::$(cat VERSION)

      - name: Docker image Build
        id: build-image
        run: |
          docker build \
            -f Dockerfile \
            -t ${{ steps.set-var.outputs.ECR_REGISTRY }}/${{ steps.set-var.outputs.ECR_REPOSITORY }}:${{ steps.set-var.outputs.IMAGE_TAG }} .

      - name: Workload Test
        id: test
        run: |
          docker run \
            --rm \
            ${{ steps.set-var.outputs.ECR_REGISTRY }}/${{ steps.set-var.outputs.ECR_REPOSITORY }}:${{ steps.set-var.outputs.IMAGE_TAG }} \
            /root/.local/bin/pytest -v

      - name: Docker image Push
        id: push-image
        run: |
          docker push ${{ steps.set-var.outputs.ECR_REGISTRY }}/${{ steps.set-var.outputs.ECR_REPOSITORY }}:${{ steps.set-var.outputs.IMAGE_TAG }}

      - name: Make zip file
        run: |
          zip -j ./$GITHUB_SHA.zip \
            deploy/appspec.yml \
            deploy/scripts/kill_process.sh \
            deploy/scripts/run_process.sh \
            deploy/docker-compose.yml
        shell: bash

      - name: Upload to S3
        env:
          REGION: ap-northeast-2
          S3_NAME: jenkins-slave-artifact-codebuild-s3
        run: aws s3 cp --region ${REGION} ./$GITHUB_SHA.zip s3://${S3_NAME}/$GITHUB_SHA.zip

  cd:
    needs: [ci]
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2

      - name: Create codeDeploy deployment
        env:
          REGION: ap-northeast-2
          CODEDEPLOY_NAME: demo-codedeploy-app
          CODEDEPLOY_GROUP_NAME: dev-codedeploy-group
          S3_NAME: jenkins-slave-artifact-codebuild-s3
        run: |
          aws deploy create-deployment \
            --application-name ${CODEDEPLOY_NAME} \
            --deployment-group-name ${CODEDEPLOY_GROUP_NAME} \
            --region ${REGION} \
            --s3-location bucket=${S3_NAME},bundleType=zip,key=$GITHUB_SHA.zip \
            --file-exists-behavior OVERWRITE \
```
