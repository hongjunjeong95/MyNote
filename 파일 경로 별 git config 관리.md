# 파일 경로 별 git config 관리

- 매번 저장소 받을때마다 local config ( name, email ) 를 회사 계정으로 바꾸기 귀찮고 까먹는 분들을 위해 준비했습니다.
- 저장소 별 config는 ... 앞으로 테스크 관리쪽에서 커밋과 해당 커밋의 author를 통해서 어떤 사람이 어떤 테스크를 얼마의 시간동안 했는지 모니터링 할 예정이므로 **필수** 입니다.

`~/.config` 아래에 `git` 폴더를 생성합니다.

```shell
cd ~/.config
mkdir git
cd git
vi .gitconfig-onhyang

### 다음과 같이 자신의 username / email 입력!
[user]
    name = yeastlen-hongjun
    email = hongjunjeong95@yeastlen.ai
```

```shell
vi ~/.gitconfig

### 각자 자신이 따로 관리하는 yeastlen 폴더 경로, 만약 관리를 따로 안한다면 하시는걸 추천드립니다.
[includeIf "gitdir:~/Repository/yeastlen/"]
  path = ~/.config/git/.gitconfig-onhyang
```