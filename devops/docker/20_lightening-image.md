# Docker - Lightening Image

## 컨테이너 레이어 수 줄이기 & 꼭 필요한 패키지 및 파일만 추가

```dockerfile
#
# slacktee
#
# build:
#   docker build --force-rm -t slacktee .
# run:
#   docker run --rm -it --name slacktee slacktee
#

FROM alpine:3.14
LABEL maintainer="FastCampus Park <fastcampus@fastcampus.com>"
LABEL description="Simple utility to send slack message easily."

# Install needed packages
RUN \
  apk add --no-cache bash curl git && \
  git clone https://github.com/course-hero/slacktee /slacktee && \
  apk del --no-cache git
RUN chmod 755 /slacktee/slacktee.sh

# Run
WORKDIR /slacktee
ENTRYPOINT ["/bin/bash", "-c", "./slacktee.sh"]
```

* 최대한 **RUN** 지시어 수를 줄이자. &&를 사용해서 여러 명령어들을 하나의 지시어에서 사용할 수 있다. 즉 세 가지 명령어를 하나의 레이어로 합쳤다.
* 도커 이미지 상에서는 패키지에 대한 캐시가 필요 없다. 그렇기 때문에 `--no-cache`를 이용해서 캐시를 제거하자.
* git을 다 사용했다면 불필요한 패키지이기 때문에 `ap del --no-cache`로 캐시를 남기지 않고 `git`을 삭제한다.

## 경량 베이스 이미지 선택

```dockerfile
#
# nodejs-server
#
# build:
#   docker build --force-rm -t nodejs-server .
# run:
#   docker run --rm -it --name nodejs-server nodejs-server
#
# FROM node:16-slim
FROM node:16-alpine
LABEL maintainer="FastCampus Park <fastcampus@fastcampus.com>"
LABEL description="Simple server with Node.js"

# Create app directory
WORKDIR /app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "node", "server.js" ]

```

*  `FROM`에서 `node:16`대신 `node:16-slim`이나 `node:alpine`을 사용했다.

![image](https://user-images.githubusercontent.com/33750210/146695247-050cf9cc-7486-464a-bae9-f442259cd0e9.png)

각각의 용량을 살펴보면 ''기본 node' > slim > alpine 순으로 용량이 경량화 되는 것을 확인할 수 있다.

## 멀티 스테이지 빌드 사용

```dockerfile
#
# nodejs-server
#
# build:
#   docker build --force-rm -t nodejs-server .
# run:
#   docker run --rm -it --name nodejs-server nodejs-server
#

FROM node:16-alpine AS base
LABEL maintainer="FastCampus Park <fastcampus@fastcampus.com>"
LABEL description="Simple server with Node.js"

# Create app directory
WORKDIR /app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./


FROM base AS build
RUN npm install
# If you are building your code for production
# RUN npm ci --only=production


FROM base AS release
COPY --from=build /app/node_modules ./node_modules
# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "node", "server.js" ]
```

위와 같은 멀티 스테이지 전략을 사용하면 **build**에 필요한 패키지 의존성들을 **release** 단계에서 제거할 수 있다.
