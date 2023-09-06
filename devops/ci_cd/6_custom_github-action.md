# Custom Github Action

## Slack

1. api.slack.com

2. Create an app

   <img src="https://user-images.githubusercontent.com/33750210/146851016-d5442575-52c8-4cbe-9b7a-d9c1785d9432.png" alt="image" style="zoom:40%;" />

3. From an scratch

   <img src="https://user-images.githubusercontent.com/33750210/146851071-17ee4b0f-1bbf-45e8-9828-3642ad3fe1e2.png" alt="image" style="zoom:50%;" />

4. Name app & choose workspace

   <img src="https://user-images.githubusercontent.com/33750210/146851140-3eb90e7d-812d-4682-b6fd-c454974363c4.png" alt="image" style="zoom:50%;" />

   1. App Name
   2. Pick a workspace
   3. Create App

5. Select Incoming Webhooks

   <img src="https://user-images.githubusercontent.com/33750210/146851262-9ac9c877-1af1-43a2-9747-5c7acc304672.png" alt="image" style="zoom:43%;" />

6. Activate Incoming Webhooks

   <img src="https://user-images.githubusercontent.com/33750210/146851364-2b5b88c0-1089-449b-8173-8b912c0a79a4.png" alt="image" style="zoom:40%;" />

   1. Click **On**
   2. 해당 curl request로 슬랙에 메시지를 보낸다.

7. Add New Webhook to Workspace

   <img src="https://user-images.githubusercontent.com/33750210/146851521-36a24a2f-8d61-4967-9764-075eb2d1220f.png" alt="image" style="zoom:50%;" />

8. 채널 연결

   <img src="https://user-images.githubusercontent.com/33750210/146851656-70d82f15-640f-4379-bbe1-49a847c16c71.png" alt="image" style="zoom:50%;" />

9. 채널을 연결하고 난 후 얻게 된 url을 복사

   <img src="https://user-images.githubusercontent.com/33750210/146851755-189daf7d-9f47-444f-9d5d-e55fb8454cb4.png" alt="image" style="zoom:50%;" />

## Slack token을 Github에 secret으로 등록

1. 슬랙 URL에서 services/ 뒤에 오는 토큰 값을 복사

   <img src="https://user-images.githubusercontent.com/33750210/146852006-e25bbb15-de35-45d3-a32f-f48d85094e5b.png" alt="image" style="zoom:60%;" />

2. Github Secret에 **SLACK_TOKEN**으로 등록

   ![image](https://user-images.githubusercontent.com/33750210/146852122-5b757570-7528-415d-abdb-16f7c04448ec.png)

## Your Proejct Repository slack.yml

```yaml
# .github/workflows/slack.yml

name: Send Slack message

on:
  push:
    branches: [master]
    paths-ignore:
      - '.gitignore'
      - '.dockerignore'
      - 'README.md'
  #    paths:
  #      - '**.py'
  pull_request:
    branches: [master]

jobs:
  slack:
    runs-on: ubuntu-latest
    steps:
      - name: Send slack message
        uses: dev-chulbuji/devops_custom_actions@master
        with:
          args: slack
        env:
          SLACK_TOKEN: ${{ secrets.SLACK_TOKEN }}
          SLACK_MESSAGE: Push event!!
```



## Custom Action Repository

### action.yml

```yaml
name: 'Devops custom actions'
description: 'Fastcampus devops custom actions'
author: "DJ"

runs:
  using: 'docker'
  image: 'Dockerfile'
```

git-action이 들어오면 docker를 사용하고 이미지로는 Dockerfile을 사용한다.

### Dockerfile

```dockerfile
FROM alpine

RUN apk update && apk upgrade && apk add curl

ADD entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

slack 메시지만 보낼 것이기 때문에 alpine 이미지를 사용하고 curl을 설치한 후 entrypoint로 `entrypoint.sh`를 사용한다.

### entrypoint.sh

```sh
#!/bin/sh

CMD="$1"

__slack() {
  if [ -z "${SLACK_TOKEN}" ]; then
    echo "SLACK_TOKEN must be set"
    exit 1
  fi

  if [ -z "${SLACK_MESSAGE}" ]; then
    echo "SLACK_MESSAGE must be set"
    exit 1
  fi

  curl \
    -XPOST \
    -H "Content-type: application/json" \
    --data "{\"text\": \"${SLACK_MESSAGE}\"}" \
    "https://hooks.slack.com/services/${SLACK_TOKEN}"
}

if [ -z "${CMD}" ]; then
  exit 1
fi

echo "[${CMD}] is running"

case "${CMD}" in
  slack)
    __slack
    ;;
  *)
    exit 1
esac
```

인자로 slack이 들어오면 _slack함수를 실행한다. 환경변수로 SLACK_MESSAGE와 SLACK_TOKEN가 등록되어 있어야 한다.



## Custom Github Action의 사용목적

여러 프로젝트가 있을 때 각각이 모두 슬랙을 보내는 로직을 구성하면 확장성이 떨어진다. 왜냐하면 똑같은 로직을 짜고 슬랙 토큰도 모두 주입해야 하기 때문이다. 그렇기 때문에 Custom Action 프로젝트에서 슬랙을 보내는 로직을 담당하고 나머지 프로젝트가 슬랙을 보내는 프로젝트에 명령만 내리면 된다. 
