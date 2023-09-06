# kubefwd 이용하여 로컬에서 쿠버네티스 서비스 접속

**Ref**

* [kubefwd](https://github.com/txn2/kubefwd)

## kubefwd 설치

MacOS

```sh
brew install txn2/tap/kubefwd
```

Ubuntu

```
wget https://github.com/txn2/kubefwd/releases/download/1.22.0/kubefwd_amd64.deb
sudo dpkg -i kubefwd_amd64.de
```



## Commands

- `kubefwd svc --all-namespaces` : 모든 네임 스페이스들로부터 모든 서비스들을 가져다가 포트 포워딩을 진행한다. 하지만 에러 메시지를 발생한다. 운영체제에 권한이 필요한 포트 범위에 대해서 포트 포워딩이 진행되기 때문에 sudo 권한이 필요하다.
  - `sudo -E kubefwd svc --all-namespaces` : sudo 명령어를 사용해야 하며 Linux에서는 `-E` 옵션까지 줘야 한다.
