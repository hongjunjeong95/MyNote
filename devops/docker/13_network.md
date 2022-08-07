# Docker - network

## 컨테이너 포트 노출

```shell
docker run -p [HOST IP:PORT]:[CONTAINER PORT] [container]
# nginx 컨테이너의 80번 포트를 호스트 모든 IP의 80번 포트와 연결하여 실행 $ docker run -d -p 80:80 nginx
docker run -d -p 80:80 nginx
# nginx 컨테이너의 80번 포트를 호스트 127.0.0.1 IP의 80번 포트와 연결하여 실행 $ docker run -d -p 127.0.0.1:80:80 nginx
docker run -d -p 127.0.0.1:80:80 nginx
# nginx 컨테이너의 80번 포트를 호스트의 사용 가능한 포트와 연결하여 실행 $ docker run -d -p 80 nginx
docker run -d -p 80 nginx
```

## Expose VS Publish

```shell
# expose 옵션은 그저 문서화 용도
docker run -d --expose 80 nginx
# publish 옵션은 실제 포트를 바인딩 $ docker run -d -p 80 nginx
docker run -d -p 80 nginx
```

## 도커 네트워크 드라이버

**Ref**

* [네트워크 드라이버](https://docs.docker.com/network/#network-drivers)

<img src="https://user-images.githubusercontent.com/33750210/146661688-1c0171ae-41fe-4264-be3b-2b523d3045b3.png" alt="image" style="zoom:45%;" />

<span style="font-size:1.9rem">**Docker Networking**</span>

<img src="https://user-images.githubusercontent.com/33750210/146662921-97205564-9f57-4cf1-a41d-695ac577d782.png" alt="image" style="zoom:50%;" />

```shell
# 기본적으로 만들어져있는 네트워크 목록 확인
docker network ls
```

* overlay : 여러 서버들이 존재한다고 했을 때 각각의 서버에 있는 컨테이너들을 연결시키는 가상의 네트워크다. 이 네트워크는 멀티 호스트로 동작하기 때문에 컨테이너를 여러 클러스터 시스템에서 동작시키는 오케스트레이션 시스템(쿠버네티스)에서 많이 쓰는 네트워크 드라이버다.
  * 오케스트레이션 시스템으로 **docker swarm**이 있다.

### none networking

```sh
#!/usr/bin/env sh

docker run -i -t --net none ubuntu:focal
```

**사용할 때**

* 해당 컨테이너가 네트워크 기능이 필요없을 때
* 커스텀 네트워킹을 사용해야 하는 니즈가 있을 때

**docker inspect 정보**

* "IPAddress" : ""
* "Bridge" : ""
* "DriverOpts" : null

`apt update` 명령어를 실행해도 네트워크 오류로 명령어가 실패한다.

### host networking

```sh
#!/usr/bin/env sh

docker run -d --network=host grafana/grafana
```

도커가 제공해주는 가상 네트워크를 사용하는 것이 아니라 직접 호스트에 붙어가지고 사용한다. 그래서 포트 바인딩을 하지 않더라도 호스트 네트워크를 사용하기 때문에 바로 접속을 할 수 있게 된다. 그래서 `docker ps`를 해도 PORTS 목록에서 포트 바인딩이 보이지는 않는다.

위 셸 파일에서 grafana를 사용하는데 grafana는 기본적으로 3000 번 포트를 사용한다. 포트 바인딩을 하지 않더라도 `curl localhost:3000`을 하면 정상적인 응답이 온다.

**docker inspect 정보**

* "IPAddress" : ""
* "Bridge" : ""
* "DriverOpts" : null
* "Hostname" : ec2의 host name

`apt update` 명령어를 실행해도 네트워크 오류로 명령어가 실패한다.

### bridge networking

```sh
#!/usr/bin/env sh

docker network create --driver=bridge lewis

docker run -d --network=lewis --net-alias=hello nginx
docker run -d --network=lewis --net-alias=grafana grafana/grafana
```

docker0(default)을 사용하는 것이 아니라 user-defined 브리지 네트워크를 사용할 것이다.

`docker network create	` 명령어를 통해서 `lewis`라는 브리지 네트워크를 새로 만든다.

* --net-alias : 이 옵션은 브리지 네트워크에서만 사용할 수 있는 옵션인데, 브리지 네트워크 안에서 해당 value 도메인 이름(hello or grafana)으로 컨테이너 ip를 search할 수 있도록 내부 도메인에 저장을 해준다.

<img src="https://user-images.githubusercontent.com/33750210/146662679-7c7c9212-e4cd-4bcd-b20c-842fc597087f.png" alt="image" style="zoom:50%;" />

![image](https://user-images.githubusercontent.com/33750210/146662707-dcae32be-edb4-446d-804f-0d4dc8991650.png)

셸 파일을 실행하면 **lewis** 네트워크가 브리지로 정상적으로 생성되고 nginx와 grafana가 정상적으로 생성되었다.

<img src="https://user-images.githubusercontent.com/33750210/146662760-2f585cff-9b50-4d51-bbe7-424174a234f3.png" alt="image" style="zoom:50%;" />

* `--net-alias=hello`로 nginx 도메인 이름을 **hello** 하였기 때문에 같은 브리지 네트워크에 있는 grafana가 wget을 이용하여 hello로 라는 이름으로 index.html 파일을 가져올 수 있었다.

## ifconfig

<img src="https://user-images.githubusercontent.com/33750210/146662832-11e57a2f-b38b-4902-ad73-555524ea3562.png" alt="image" style="zoom:40%;" />

* br : lews 브리지 네트워크
* docker0 : 브리지 네트워크
* ens5 : 호스트의 eth다. 즉 ec2 instance의 eth0이다.

<img src="https://user-images.githubusercontent.com/33750210/146662852-efb2dd5f-1063-4bf7-8452-429f290b8824.png" alt="image" style="zoom:50%;" />

컨테이너 개수(2개) 만큼 가상의 veth 네트워크가 생성되었다. 
