# Docker - Image Compress

## 이미지 압축파일로 저장

```shell
# docker save -o [OUTPUT-FILE] IMAGE
# ubuntu:focal 이미지를 ubuntu_focal.tar 압축 파일로 저장
docker save -o ubuntu_focal.tar ubuntu:focal
```

이미지를 tar 압축파일로 저장

## 이미지 압축에서 불러오기

```shell
# docker load -i [INPUT-FILE]
# ubuntu_focal.tar 압축 파일에서 ubuntu:focal 이미지 불러오기
docker load -i ubuntu_focal.tar
```

이미지를 tar 압축파일로부터 불러온다.

인터넷이 안 되는 환경이나 다른 사람한테 인터넷 퍼블릭 레포지토리를 거치지 않고 이미지를 전달할 때 유용하게 사용할 수 있다.
