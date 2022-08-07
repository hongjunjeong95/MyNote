# Docker - Entrypoint and Command

## Entrypoint

도커 컨테이너가 실행할 때 고정적으로 실행되는 스크립트 혹은 명령어. 생략할 수 있으며, 생략될 경우 커맨드에 저징된 명령어로 수행

```shell
docker run --entrypoint echo ubuntu:focal heloo world
docker ps -a
```

![image](https://user-images.githubusercontent.com/33750210/146629480-dcdecde1-1bd9-43b9-9f8e-ddefbe5fce4e.png)

COMMAND로 "echo heloo world"가 등록되어 있다.

```shell
docker inspect 7464
```

<img src="https://user-images.githubusercontent.com/33750210/146629509-6ff3d823-d7ce-453b-8ddf-4782ae357dd3.png" alt="image" style="zoom:50%;" />

상세 정보를 살펴보면 **Entrypoint**로 "echo"가 set되었고 **Cmd**로 ["heloo", "world"]가 추가 되었다. 

## Command

도커 컨테이너가 실행할 때 수행할 명령어 혹은 엔트리포인트에 지정된 명령어에 대한 인자 값
