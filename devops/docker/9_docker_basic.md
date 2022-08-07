# Docker Basic

## Commands

* `docker run -it ubuntu:focal` : 이 명령어로 ubuntu bash 쉘에 접근할 수 있다. `exit` 명령어로 컨테이너를 빠져나올 수 있지만 동시에 컨테이너가 종료된다. 컨테이너를 종료하지 않고 빠져 나오고 싶다면 `ctrl + p, q` key를 누른다.

## 컨테이너 상태 확인

* `docker ps` : 실행 중인 컨테이너 상태 확인
* `docker ps -a` : 전체 컨테이너 상태 확인
* `docker inspect <container_id>` : 컨테이너 상제 정보 확인. 컨테이너를 운영함에 있어서 문제가 생기면 해당 명령어로 상세 정보들을 확인한다.

## 컨테이너 일시중지 및 재개

* `docker pause <container_id>` : 컨테이너 일시중지
* `docker unpause <container_id>` : 컨테이너 재개

## 컨테이너 종료

* `docker stop <container_id>` : 컨테이너 종료(SIGTERM 시그널 전달)
* `docker stop $(docker ps -a -q)` : 모든 컨테이너 종료
* `docker kill <container_id>` : 컨테이너 강제 종료(SIGKILL 시그널 전달)

## 컨테이너 삭제

* `docker rm <container_id>` : 컨테이너 삭제(실행중인 컨테이너 불가)
* `docker rm -f <container_id>` : 컨테이너 강제 종료 후 삭제(SIGKILL 시그널 전달)
* `docker run --rm ...` : 컨테이너 실행 종료 후 자동 삭제
* `docker container prune` : 중지된 모든 컨테이너 삭제
