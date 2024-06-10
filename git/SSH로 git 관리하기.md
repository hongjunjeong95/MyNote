# SSH로 git 관리하기

## 1. ssh key 생성

`cd ~/.ssh`
`ssh-keygen -t rsa -C "~~~@yeastlen.ai"`

### ‼️‼️‼️중요‼️‼️‼️

##### 명령어 입력 후 hongjun_rsa(예시) 라고 파일 이름을 명시해줘야

##### 기존 key가 안 덮어씌워집니다.



## 2. 생성된 퍼블릭 키를 자신의 회사 계정의 github에 등록합니다 ( company 계정으로 로그인)

우측 상단 프로필 사진 클릭 => settings => SSH and GPG keys => SSH keys => New SSH Key => 퍼블릭 키(ssh-keygen으로 마든 .pub 파일) 등록



## 3. Could not open a connection to your authentication agent 에러

git push 할 때 위와 같은 error가 뜨면

```shell
# ssh-agent를 시작
$ eval $(ssh-agent)

# hongjun_rsa 파일의 권한이 너무 열려 있다고 경고가 뜰 경우
$ chmod 400 ~/.ssh/hongjun_rsa

# 개인키를 등록
$ ssh-add ~/.ssh/hongjun_rsa
```



## 4. 배포시 원격 저장소를 찾지 못한다면 `ssh-add ~/.ssh/yeastlen_rsa` 후 배포

그래도 안되면 `ssh-add -D ~/.ssh/hongjun_rsa` 로 다 지우고 다시 `ssh-add ~/.ssh/hongjun_rsa`