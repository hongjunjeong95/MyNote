# CircleCI

## 특징

* Project : code repository in VCS
* Pipelines
  * Full set of process
  * .circleci/config.yml
* Workflows
  * list of jobs
  * run jobs concurrently, sequentially, on a schedule
  * manual gate using an approval job
* Jobs
  * set of Steps
  * run in executor
* Steps
  * collections of commands
  * built-in steps(checkout,..), custom command(run, ..)
  * Define Commands outside jobs -> Reusable across jobs
* Executors
  * runner for running job
  * container, vm (linux, windows, macOS)
* Orbs
  * reusable snippets of code
  * orbs repository(Github의 marketplace같은 곳)

## Usage

<img src="https://user-images.githubusercontent.com/33750210/146854425-7a1e8410-cbbc-4d17-83e4-8add8cea99a0.png" alt="image" style="zoom:50%;" />

로그인을 하고 나서 레포지토리를 선택한다.

### .circleci/config.yml

```yaml
version: 2.1

orbs:
  aws-cli: circleci/aws-cli@2.0.3
  aws-ecr: circleci/aws-ecr@7.2.0
  aws-s3: circleci/aws-s3@3.0.0

commands:
  test:
    steps:
      - checkout
      - run: |
          aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ECR_ACCOUNT_URL
      - run: |
          docker run --rm \
            $AWS_ECR_ACCOUNT_URL/$APP_NAME:$(cat VERSION) \
            /root/.local/bin/pytest -v
  upload-s3:
    steps:
      - checkout
      - run: |
          zip -j ./$CIRCLE_BUILD_NUM.zip \
            deploy/appspec.yml \
            deploy/scripts/kill_process.sh \
            deploy/scripts/run_process.sh \
            deploy/docker-compose.yml
      - aws-s3/copy:
          from: ./$CIRCLE_BUILD_NUM.zip
          to: 's3://jenkins-slave-artifact-codebuild-s3'
  create-codedeploy-deployment:
    steps:
      - run: |
          aws deploy create-deployment \
            --application-name demo-codedeploy-app \
            --deployment-group-name dev-codedeploy-group \
            --region $AWS_REGION \
            --s3-location bucket=jenkins-slave-artifact-codebuild-s3,bundleType=zip,key=$CIRCLE_BUILD_NUM.zip \
            --file-exists-behavior OVERWRITE


#executors:
#  circlecipython:
#    docker:
#      - image: circleci/python

jobs:
  test:
    executor: aws-cli/default
    steps:
      - aws-cli/setup:
          profile-name: default
      - setup_remote_docker:
          docker_layer_caching: true
          version: 20.10.7 # docker version
      - test
  deploy:
    executor: aws-cli/default
    steps:
      - aws-cli/setup:
          profile-name: default
      - upload-s3
      - create-codedeploy-deployment

workflows:
  cicd:
    jobs:
      - aws-ecr/build-and-push-image:
          name: build-and-push-image
          context: AWS
          checkout: true
          tag: $(cat VERSION)
          repo: demo
          filters:
            branches:
              only: master
      - test:
          requires:
            - build-and-push-image
          context: AWS
      - deploy:
          requires:
            - test
          context: AWS
```

($)가 붙는 context는 circleci에서 key-value로 선언할 수 있다.



















