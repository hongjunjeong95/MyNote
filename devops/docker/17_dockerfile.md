# Docker - Image Build

**Ref**

* https://docs.docker.com/engine/reference/builder/

## 환경 변수

```dockerfile
ENV FOO=/bar
WORKDIR ${FOO}
```

도커파일의 환경변수는 호스트 환경변수가 아니라 컨테이너 환경변수다.

## ARG

```dockerfile
USER ${user1:-some_user}
ARG user1=someone
ARG user2
USER $user1
```

ENV대신 ARG를 사용할 수 있다. 키-밸류 형식을 사용하고 default 값을 줘도 되고 안 줘도 된다.

```shell
docker build --build-arg user2=what_user
```

명령어로 `--build-arg` 옵션을 줘서 변수에 value를 줄 수도 있다.

### Scope

```dockerfile
USER ${user:-some_user}
ARG user=foo
USER $user
```

ARG로 `user=foo` 값을 줬어도 호이스팅이 일어나지 않기 때문에 최상단 USER에서는 user가 -some_user로 할당되고 마지막 USER에서 `user=foo`가 할당된다.

### ENV Overwrite

```dockerfile
ARG CONT_IMG_VER
ENV CONT_IMG_VER=v1.0.0
RUN echo $CONT_IMG_VER
```

ARG와 ENV가 똑같은 변수명을 가질 때 ENV가 ARG를 덮어씌운다.
