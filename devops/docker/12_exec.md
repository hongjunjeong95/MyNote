# Docker - exec

해당 컨테이너에 문제가 생겼을 때 원인 분석을 위하여 실행 중인 해당 컨테이너에 shell로 접근할 수 있다(대신 sh, shell, bash가 존재해야 한다.).

```shell
# my-nginx 컨테이너에 Bash 셸로 접속하기
docker exec -it my-nginx bash

# my-nginx 컨테이너에 환경변수 확인하기
docker exec my-nginx env
```
