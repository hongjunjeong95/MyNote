# Docker - Log

## stdout / stderr

<img src="https://user-images.githubusercontent.com/33750210/146672523-6b2c80ab-8dac-4baa-abaa-d57abdcdc4e2.png" alt="image" style="zoom:40%;" />

 도커 컨테이너에서 로그를 다루기 위해서는 앱에서 로그를 표준출력과 표준에러로 내보내는 것을 표준으로 삼아야 한다. 출력되는 로그를 도커가 쌓아가지고 logging driver가 처리할 수 있도록 한다. 도커는 다양한 logging driver를 제공하는데 가장 기본적으로 사용되는 드라이버는 `json-file`이라는 형태로 한 줄의 json 하나로 구성되는 로그 파일이다.

## 로그 확인하기

```shell
# 전체 로그 확인
docker logs <container>

# 마지막 로그 10줄 확인
docker logs --tail 10 <container>

# 실시간 로그 스트림 확인
docker logs -f <container>

# 로그마다 타임스탬프 표시
docker logs -f -t <container>
```

## 호스트 운영체제의 로그 저장 경로

```shell
cat /var/lib/docker/containers/${container_id}/${container_id}-json.log
```

log driver = json-file로 했을 때만 유효하다. inline형태의 json을 파일로 남긴다. 이 파일이 호스트 운영체제에 저장된다. 

## 로그 용량 제한하기

```shell
# 한 로그 파일 당 최대 크기를 3Mb로 제한하고, 최대 로그 파일 5개로 로테이팅.
docker run \
-d \
--log-driver=json-file \
--log-opt max-size=3m \
--log-opt max-file=5 \
nginx
```

도커를 OS에 설치하게 되면 로그 용량에 대한 설정이 되어 있지 않다. 

* 컨테이너 단위로 로그 용량 제한을 할 수 있지만, 도커 엔진에서 기본 설정을 진행할 수도 있다. (OS에서 필수 설정)

## 도커 로그 드라이버

![image-20211219201405975](/Users/hongjun/Library/Application Support/typora-user-images/image-20211219201405975.png)

json-file이라는 로그 드라이버를 사용 => inline 형태의 json 파일로 호스트 OS에 저장 => 호스트 OS 상에 로그 에이전트(Log shipper)를 설치 => 해당 에이전트로 중앙화된 로그 관리 시스템에 전송(Elastic search, cloudwathc, splunk, graylog 등) => 중앙화된 로그 시스템에서 컨테이너 로그를 쌓는다. 

























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

* 도커가 제공하는 볼륨 관리 기능을 활용하여 데이터를 보존합니다.

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
