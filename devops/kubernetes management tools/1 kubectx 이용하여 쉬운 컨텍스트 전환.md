# kubectx 이용하여 쉬운 컨텍스트 전환

**Ref**

* [kubectx](https://github.com/ahmetb/kubectx)

## kubectx 설치

MacOS

```
brew install kubectx
```

Ubuntu

```
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```



## Commands(kubectl)

- `kubectl cofnig` : sub command 확인
- `kubectl config current-context` : 지금 바라보고 있는 context 확인
- `kubectl config get-contexts` : 존재하는 컨텍스트 정보들을 목록으로 출력할 수 있다.
- `kubectl cluster-info` : 지금 어떤 context를 가리키고 있는지 확인할 수 있다.

## Commands(kubectx)

- `kubectx <context>` : context 전환
- `kubect -` : 이전 context로 전환
- `kubens` : 클러스터에 존재하는 namespae 목록을 출력할 수 있다.
- `kubens <namespace>` : default로 네임스페이스가 되어 있어서 `kubectl get pod`를 할 때 `-n` 옵션으로 namespace를 지정해줘야 한다. 하지만 `kubens <namespace>` 를 사용하면 해당 네임스페이스로 디폴트 설정이 되어서 kubectl 명령어를 사용할 때 -n 옵션을 주지 않아도 된다.

컨텍스트는 3가지 정보의 조합이다. Cluster, user and namespace다. 어떤 클러스터에 어떤 사용자로 기본 네임 스페이스를 무엇으로 사용할지에 대한 설정이다.

컨텍스트 정보는 `cat ~/.kube/config` 파일을 열어보면 확인할 수 있다.

