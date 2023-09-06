# Docker - Docker Daemon Debugging

## Docker Daemong Debugging Commands

```shell
# 해당 시스템에 대한 정보 출력
docker system info

# 스트리밍 형식으로 새롭게 발생하는 도커 이벤트들이 흘러들어온다.
docker system events
docker events

# docker log들을 확인할 수 있음
journal -u docker
 
# `df -h` 디스크 사용량을 살펴볼 수 있는 것처럼 docker system df도 도커의 디스크 사용량을 살펴볼 수 있다.
docker system df

# 더 자세하게 디스크 사용량을 살펴봄
docker system df -v

# 중지된 컨테이너, 사용되지 않는 네트워크, dangling images와 dangling build cache를 모두 제거한다.
docker system prune

# 각각의 컨테이너별로 CPU, 메모리와 네트워크 I/O를 사용하는지 확인할 수 있다.
docker stats
```



