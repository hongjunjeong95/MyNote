# k9s를 이용하여 CLI로 쿠버네티스 클러스터 관리

**Ref**

* [k9s](https://github.com/derailed/k9s)
* [k9s key bindings](https://github.com/derailed/k9s#key-bindings)

## k9s 설치

MacOS

```sh
brew install k9s
```

Ubuntu

```
curl -sS https://webinstall.dev/k9s | bash
```



## Commands

- `k9s` : k9s 콘솔 접속
- `:` => 콜론으로 명령어 칠 수 있음
  - context : context 보기
  - deployment : deployment 보기
  - service : service 보기
  - pod : pod 보기
  - namespace : namespace 보기
- `/` => 필터창을 킨다.
  - 아무 옵션없이 문자열을 쓰면 문자열이 포함된 name을 출력한다.
  - `!` : 문자열이 not인 name을 출력
  - `-l app=<name>` : `-l` 옵션은 label selector인데, `app=<name>` 레이블을 가지는  파드를 찾는다.
- `?` : 어떤 shortcut을 쓸 수 있는지 알 수 있음
- `0` : 모든 네임스페이스에 대한 파드 확인
- `1` : 디폴트 네임스페이스에 대한 파드 확인
- `xray <resource_type>` : `<resource_type>`이 deployment일 경우 클러스터 상에 존재하는 모든 deployment 객체를 트리 구조로 보여준다.
- `popeye` : 클러스터의 상태가 잘 설정이 되어 있는지 전반적인 체크를 한다 각각의 컴포넌트 별로 점수를 매기는데, 잘못된 게 있으면 빨간색으로 나오고 Enter를 누르면 어떤 조치를 취할 수 있는지 알려준다.
