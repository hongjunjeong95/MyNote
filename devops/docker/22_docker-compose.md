# Docker - Docker Compose

**Ref**

* [docker-compose](https://docs.docker.com/compose/compose-file/compose-file-v3/)

## docker-compose 명령어 : 프로젝트 목록

```shell
# 실행중인 프로젝트 목록 확인
docker-compose ls

# 전체 프로젝트 목록 확인
docker-compose ls -a 
```

## docker-compose 명령어 : 실행 및 종료

```shell
# Foreground로 도커 컴포즈 프로젝트 실행
docker-compose up

# Background로 도커 컴포즈 프로젝트 실행
docker-compose up -d

# 프로젝트 이름 my-project로 변경하여 도커 컴포즈 프로젝트 실행
docker-compose -p my-project up -d

# 프로젝트 내 컨테이너 및 네트워크 종료 및 제거
docker-compose down

#프로젝트 내 컨테이너,네트워크 및 볼륨 종료 및 제거
docker-compose down -v
```

## docker-compose 명령어 : 서비스 확장

```shell
# web 서비스를 3개로 확장
docker-compose up --scale web=3
```

docker-compose.yml에서 container_name과 port 맵핑이 된다면 scaling이 불가능하니 이 두 명령어는 추가하지 않는다.

## docker-compose 명령어

```shell
# 프로젝트 내 서비스 로그 확인
docker-compose logs

# 프로젝트 내 컨테이너 이벤트 확인
docker-compose events

# 프로젝트 내 이미지 목록
docker-compose images

# 프로젝트 내 컨테이너 목록
docker-compose ps

# 프로젝트 내 실행중인 프로세스 목록
docker-compose top
```

## docker-compose.yml

```yaml
version: '3.9'

services:
  db:
    image: mysql:5.7
    volumes:
    - db:/var/lib/mysql
    restart: always
    environment:
    - MYSQL_ROOT_PASSWORD=wordpress
    - MYSQL_DATABASE=wordpress
    - MYSQL_USER=wordpress
    - MYSQL_PASSWORD=wordpress
    networks:
    - wordpress

  wordpress:
    depends_on:
    - db
    image: wordpress:latest
    ports:
    - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    networks:
    - wordpress

volumes:
  db: {}

networks:
  wordpress: {}
```

* depends_on : 해당 서비스가 실행되고난 후 실행할 것을 보장한다. 하지만 해당 서비스가 준비되었음을 보장하지는 않는다.

## Use Cases

* 로컬 개발 환경 구성 : 특정 프로젝트의 로컬 개발 환경 구성 목적으로 사용 프로젝트의 의존성(Redis, MySQL, Kafka 등)을 쉽게 띄울 수 있음
*  자동화된 테스트 환경 구성 : CI/CD 파이프라인 중 쉽게 격리된 테스트 환경을 구성하여 테스트 수행 가능
* 단일 호스트 내 컨테이너를 선언적 관리 : 단일 서버에서 컨테이너를 관리할 때 YAML 파일을 통해 선언적으로 관리 가능
