# Docker - Volume

## 도커 레이어 아키텍쳐

<img src="https://user-images.githubusercontent.com/33750210/146671049-8126d9fd-28d4-4621-9329-f3aff3269f7b.png" alt="image" style="zoom:40%;" />

### 호스트 볼륨

```shell
#!/usr/bin/env sh

 # 호스트의 /opt/html 디렉토리를 Nginx의 웹 루트 디렉토리로 마운트
docker run \
  -d \
  -v $(pwd)/html:/usr/share/nginx/html \
  -p 80:80 \
  nginx
```

* 호스트의 디렉토리를 컨테이너의 특정 경로에 마운트한다.
* 만약 bash 셸로 도커의 `/usr/share/nginx/html`에 파일을 생성하면 호스트 볼륨인 $(pwd)/html에도 파일이 생성된다. 왜냐하면 `/usr/share/nginx/html`은 실제로는 도커 볼륨이 아니라 호스트 볼륨이기 때문이다.

### 볼륨 컨테이너

```sh
#!/usr/bin/env sh

docker run \
  -d \
  -it \
  -v $(pwd)/html:/usr/share/nginx/html \
  --name web-volume \
  ubuntu:focal

# my-volume 컨테이너의 볼륨을 공유
docker run \
  -d \
  --name lewis-nginx \
  --volumes-from web-volume \
  -p 80:80 \
  nginx

docker run \
  -d \
  --name lewis-nginx2 \
  --volumes-from web-volume \
  -p 8080:80 \
  nginx
```

* 특정 컨테이너의 볼륨 마운트를 공유할 수 있다.
* `--volumes-from` 옵션으로 컨테이너 이름을 가리킨다. 그러면 해당 컨테이너의 볼륨을 watch한다. 위 예제에서 **web-volume**의 볼륨은 `$(pwd)/html:/usr/share/nginx/html`이다. **lewis-nginx**와  **lewis-nginx2**의 `/usr/share/nginx/html`는 web-volume의`/usr/share/nginx/html`을 따라간다.

### 도커 볼륨

```sh
#!/usr/bin/env sh

# db 도커 볼륨 생성
docker volume create --name db

docker volume ls

 # 도커의 db 볼륨을 Nginx의 웹 루트 디렉토리로 읽기 전용 마운트
docker run \
  -d \
  --name fastcampus-mysql \
  -e MYSQL_DATABASE=leweis \
  -e MYSQL_ROOT_PASSWORD=lewis \
  -v db:/var/lib/mysql \
  -p 3306:3306 \
  mysql:5.7
```

* 도커가 제공하는 볼륨 관리 기능을 활용하여 데이터를 보존한다.

* 기본적으로 `/var/lib/docker/volumes/${volume-name}/_data` 에 데이터가 저장된다.

  * 이 예제에서는 `/var/lib/docker/volumes/db/_data` 가 된다.

* volume을 삭제하고 싶다면

  ```shell
  docker rm -f `docker ps -aq`
  docker volume rm <volume name>
  ```

### 읽기전용 볼륨 연결

```sh
#!/usr/bin/env sh

# 도커의 web-volume 볼륨을 Nginx의 웹 루트 디렉토리로 읽기 전용 마운트
docker run \
  -d \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  -p 80:80 \
  --name ro-nginx \
  nginx

docker exec ro-nginx touch /usr/share/nginx/html/hello
```

* 볼륨 연결 설정에 :ro 옵션을 통해 읽기 전용 마운트 옵션을 설정할 수 있다.
* 읽기 전용 마운트는 변경이 되서는 안 되는 디렉토리나 파일을 연결할 때 진행이 된다.
  * 보통 해당 애플리케이션의 설정 마운트를 할 때 ro 옵션을 준다거나
  * 지금처럼 Nginx 웹 루트 디렉토리에 연결을 할 때는 웹의 static 파일들은 변경될 일이 없으니 :ro 플래그를 준다.

위 셸 스크립트를 실행하면

![image](https://user-images.githubusercontent.com/33750210/146672430-b69ca1d4-bd21-4ffc-a85e-f70db65c183c.png)

다음과 같이 Read-only file system 에러가 발생한다.
