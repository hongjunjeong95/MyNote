# Issues

## yarn berry로 docker를 올렸을 때 bcrypt 모듈 OS 이슈

### Cause

bcrypt는 native 모듈이므로 OS와 가깝다. 이 뜻은 OSX에서 bcrypt를 설치하면 OSX에 맞춰지기 때문에 도커(linux 환경)에 올렸을 때 작동하지 않는다.

### Solution

.dockerignore에 .yarn을 추가해서 도커가 OSX의 bcrypt를 포함시키지 않도록 해야 한다. 그리고 docker-compose 볼륨에 .yarn을 가리키지 않도록 해야 한다.

#### .dockerignore

```.dockerignore
.yarn
```

#### docker-compose.yml

```yaml
services:
  backend:
	  ...
    volumes:
      - /usr/src/app/.yarn
      - ./:/usr/src/app
```



## yarn berry로 docker를 올렸을 때 bcrypt 모듈 python 이슈

### Cause

bcrypt는 pyton으로 wrapping되었다. 보통 도커 파일 경량화를 위해 alpine 이미지를 사용하느데, 해당 이미지는 모든 것을 최소화 했기 때문에 python module이 없다. 그래서 multistaging을 통해서 build를 할 때는 python이 있는 기본 노드 이미지를 사용해서 모듈들을 설치하고 서버를 실행할 때는 alpine 이미지를 사용해야 한다.

### Solution

#### Dockerfile.dev

```dockerfile
FROM node:16 AS builder

WORKDIR /usr/src/app
COPY ./package.json ./
RUN yarn set version berry
RUN yarn plugin import workspace-tools # typescript 적용
COPY yarn.lock .yarnrc.yml ./
RUN yarn
COPY . .
CMD ["yarn", "run", "build"]

FROM node:16-alpine
WORKDIR /usr/src/app
COPY --from=builder /usr/src/app ./
CMD [ "yarn" ,"run","start:dev"]
EXPOSE 5000
```