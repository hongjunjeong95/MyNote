# Docker - Environment Variable

## Command

`docker run -e KEY=value <image>`에서 **-e 옵션** 으로 환경변수를 주입할 수 있다.

![image](https://user-images.githubusercontent.com/33750210/146661173-0534c0b1-064d-44c6-8eb3-26b6dbf62ce6.png)

`docker inspect <container_id>`로 상세정보에서 "Env"를 확인하면 MY_HOST 환경변수가 주입된 것을 확인할 수 있다.

<img src="https://user-images.githubusercontent.com/33750210/146661175-c494c3a4-2ae4-4183-b99c-0aaae9fdf62e.png" alt="image" style="zoom:50%;" />

## File Injection

#### sample.env

```env
MY_HOST=helloworld.com
MY_VAR=123
MY_VAR2=456
```

```shell
docker run -it --env-file ./sample.env ubuntu:focal env
```

`--env-file` 명령어로 env 파일을 지정해서 환경변수를 주입할 수 있다.







