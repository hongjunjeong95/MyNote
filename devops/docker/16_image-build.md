# Docker - Image Build

## 도커 이미지 구조

<img src="https://user-images.githubusercontent.com/33750210/146692317-6e78e8e1-124a-41f1-ac36-5c310f67147e.png" alt="image" style="zoom:50%;" />

`docker images inspect nginx:latest` 명령어를 쳤을 나온 레이어 정보다.

## Dockerfile 없이 이미지 생성

기존 컨테이너를 기반으로 새 이미지를 생성할 수 있다.

```shell
# docker commit [OPTIONS] CONTAINER [REPOSITORY:[:TAG]]
# ubuntu 컨테이너의 현재 상태를 my_ubuntu:v1 이미지 생성
docker commit -a lewis -m "First commit" ubuntu my_ubuntu:v1
```

* -a : author

![image](https://user-images.githubusercontent.com/33750210/146692517-d5259ec3-fd06-41c7-a817-42d0b7b2e1e4.png)

컨테이너 기반으로 도커 이미지가 생성되었다.

### 레이어 확인

**ubuntu:focal**

![image](https://user-images.githubusercontent.com/33750210/146692587-74ca3566-710f-42cc-a0d3-1ff6cc779f25.png)

#### my-ubuntu:v1

![image](https://user-images.githubusercontent.com/33750210/146692615-a4553d31-f334-4458-aeaf-9823439b255c.png) 

my-ubuntu:v1은 ubuntu:focal 이미지를 기반으로 만들어졌다. 레이어를 살펴보면 my-ubuntu:v1이 ubuntu:focal 레이어 위에 한 개 더 레이어를 쌓은 것을 확인할 수 있다.

## 빌드 컨텍스트

도커 빌드 명령 수행 시 현재 디렉토리를 빌드 컨텍스트라고 한다. Dockerfile로부터 이미지 빌드에 필요한 정보를 도커 데몬에게 전달하기 위한 목적이다.

## 빌드 옵션

* --force-rm=false: 이미지 생성에 실패했을 때도 임시 컨테이너를 삭제한다.
* --no-cache=false: 이전 빌드에서 생성된 캐시를 사용하지 않는다. Docker는 이미지 생성 시간을 줄이기 위해서 Dockerfile의 각 과정을 캐시하는데, 이 캐시를 사용하지 않고 처음부터 다시 이미지를 생성한다.
* -q, --quiet=false: Dockerfile의 RUN이 실행한 출력 결과를 표시하지 않는다.
* --rm=true: 이미지 생성에 성공했을 때 임시 컨테이너를 삭제한다.
* -t, --tag=””: 저장소 이름, 이미지 이름, 태그를 설정한다.
