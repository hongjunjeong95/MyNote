# Docker - docker hub

## 도커 허브에 로그인

<img src="https://user-images.githubusercontent.com/33750210/146694179-65ac526d-e8b4-429f-aad0-eed55c970868.png" alt="image" style="zoom:50%;" />

Account Settings

<img src="https://user-images.githubusercontent.com/33750210/146694215-fab6a15c-3ca6-4577-8572-cae112b309ba.png" alt="image" style="zoom:40%;" />

1. Security
2. New Access Token

<img src="https://user-images.githubusercontent.com/33750210/146694240-9210f4d6-b779-4ee9-9e4b-4894252091ce.png" alt="image" style="zoom:50%;" />

1. Description
2. Access permissions
3. Generate

<img src="https://user-images.githubusercontent.com/33750210/146694312-fe10d900-4c1e-4424-98a1-353876bf06c4.png" alt="image" style="zoom:40%;" />

로그인을 할 때 패스워드로 필요한 토큰이다. 복사한다.

<img src="https://user-images.githubusercontent.com/33750210/146694285-772391f3-760b-4b06-9932-b399bdc5a9ec.png" alt="image" style="zoom:50%;" />

1. `docker login -u <username>`
2. Password : 위에서 복사한 토큰을 붙여넣는다.

![image](https://user-images.githubusercontent.com/33750210/146694348-b04a8f95-dce1-4928-b4a7-f5efda1e3886.png)

`~/.docker/config.json`에 인증 키 값이 들어가 있다.

## 도커 허브에 레포지토리 생성

![image](https://user-images.githubusercontent.com/33750210/146694393-653fd7fd-0cfa-48be-a43f-64639fb76089.png)

1. Repositories
2. Create Repository

![image](https://user-images.githubusercontent.com/33750210/146694427-a3a2df06-0112-4cdd-a7ed-9cfee2dcc385.png)

1. 레포지토리 이름
2. Visibility : Private
3. Create

```shell
docker tag local-image:tagname new-repo:tagname
docker push new-repo:tagname
```

셸에서 푸시할 이미지를 태그하고 해당 태그된 이미지를 푸시한다.
