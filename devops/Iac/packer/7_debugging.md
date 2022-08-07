# Packer - Debugging

**Ref**

* [Post-Processor](https://www.packer.io/docs/post-processors)

이미지 파일을 만드는 것은 굉장히 오랜 시간이 걸린다. 프로비저닝을 하면서 5~10분 정도의 시간이 걸린다. 만약 이 때 에러가 난다면 다시 재빌드 하는데 엄청난 시간을 소요할 것이다. 이 때 디버깅을 잘 사용해서 문제점을 찾아야 한다.

Breakpoint provisioner를 사용하는 방법도 있지만 아래처럼 debug option을 줘서 디버깅을 할 수 있다.

## 디버깅

```shell
packer build -debug .
```

위 명령어로 한 단계씩 명령을 수행해 나가면서 이슈를 찾는다. 패커로 이미지를 빌드할 때 임시로 ec2 instance를 띄우기 때문에 임시 .pem 키를 생성하게 된다. 디버그로 실행을 하면 이 키가 로컬에 저장된 것을 확인할 수 있다. 사용자는 이 키를 가지고 ec2 instance에 접속할 수 있다. 디버깅을 진행하는 도중 갑자기 명령이 죽어버리면 ec2 instance에 접속해서 `cat /var/log/syslog`을 통해서 시스템 로그를 확인할 수 있고 `cat /var/log/cloud-init`도 확인하면서 문제를 파악할 수 있다. 
