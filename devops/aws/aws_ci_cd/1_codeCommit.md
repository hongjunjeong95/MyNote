# CodeCommit

CodeCommit은 Github, Gitlab과 유사하다.

## 장점

* Github와는 다르게 저장소가 암호화되어 보안적인 측면에서 우수
* 정해진 조건 내에서 무료 서비스 제공
  * 월 활성 접속계정(한달 내에 CodeCommit에 접속한 사용자 수)이 5개 이하인 경우
  * 계정 당 요청 수가 2000개 이하인 경우
  * 월 활성 접속계정이 6개 이상이 되면 계정 당 한달에 1.0 USD를 지불해야 한다.


## 환경설정

AWS Service와 통신할 때는 IAM 권한이 필요하다.

## 1. IAM 그룹 및 계정 생성

CodeCommit을 사용학 위해서는 CodeCommit 권한을 가진 IAM 계정이 필요하다. 루트 계정으로 HTTPS 접속은 보안상 위험하다고 AWS가 경고한다.

![image](https://user-images.githubusercontent.com/33750210/145493828-2e84ce44-d993-4911-81e3-ddc0a41a42c2.png)

**Create group**

<img src="https://user-images.githubusercontent.com/33750210/145494097-cf99b888-65fd-47d0-aed1-9f047b70b273.png" alt="image" style="zoom:50%;" />

User group name 설정

<img src="https://user-images.githubusercontent.com/33750210/145494116-9e5e0be2-5850-4187-9e06-2ebd6bd16f49.png" alt="image" style="zoom:40%;" />

1. **AWSCodeCommitFullAcces** 정책 부여
2. Create group

![image](https://user-images.githubusercontent.com/33750210/145494256-80935121-15aa-41a1-90e6-74a81394b6bf.png)

CodeCommit repository에 접근할 user 생성

1. Users
2. Add users

<img src="https://user-images.githubusercontent.com/33750210/145494396-01563c66-49d5-4f4f-866e-aa70075ff029.png" alt="image" style="zoom:40%;" />

* User name 작성
* Acess Key와 Password 모두 체크
* AWS Console에 접속할 수 있는 password 작성
* 계정을 할당 받은 사원이 다음 로그인 때 패스워드를 변경할 수 있도록 **Require password reset** 권한 부여
* Nest: Permissions

<img src="https://user-images.githubusercontent.com/33750210/145494580-6b355cb6-e950-4033-a3f1-b76bfc3b82ee.png" alt="image" style="zoom:40%;" />

codecommit-team에 유저 추가 후 **Next:Tags** => **Next:Review** => **Create user**

<img src="https://user-images.githubusercontent.com/33750210/145494705-f4bebfa2-f314-445e-ab38-63e3da762146.png" alt="image" style="zoom:50%;" />

Access key와 Secret key 저장

## 2. 로컬 pc간 SSH 연결

```shell
cd ~/.ssh
ssh-keygen -t rsa -f insomenia_rsa
cat ~/.ssh/insomenia_rsa.pub|pbcopy # clipboard에 ssh-public-key 복사
```

<img src="https://user-images.githubusercontent.com/33750210/145494828-603df3ae-38b0-4b57-9be6-f03b2f6b8f4f.png" alt="image" style="zoom:50%;" />

**Users**에서 user(codecommit-user-1) 선택

<img src="https://user-images.githubusercontent.com/33750210/145494994-d3c23899-9570-4cb7-ae5c-d21f30acf2f6.png" alt="image" style="zoom:43%;" />

1. **Security credentials** 선택
2. **Upload SSH public key** 선택

<img src="https://user-images.githubusercontent.com/33750210/145495211-87c005a9-b00d-4fea-9195-04d039f3b18c.png" alt="image" style="zoom:40%;" />

1. ssh-public-key 복사
2. **Upload SSH public Key**



<img src="https://user-images.githubusercontent.com/33750210/145495450-cc3121f2-bd23-4b5b-a258-ba64050f6d38.png" alt="image" style="zoom:50%;" />

SSH key ID 복사



### ~/.ssh/config 파일 작성

```
Host git-codecommit.*.amazonaws.com
 User [위에서 복사한 SSH 키 ID]
 IdentityFile ~/.ssh/[ssh-keygen으로 생성한 private-key]
```

```shell
ssh git-codecommit.ap-northeast-2.amazonaws.com
```

ssh 접속 시도

<img src="https://user-images.githubusercontent.com/33750210/145495828-09b35d94-5562-439f-9106-0768b5cb3679.png" alt="image" style="zoom:50%;" />

위와 같은 메시지가 뜨면 정상적으로 접속한 것이다.

## Create CodeCommit

![image](https://user-images.githubusercontent.com/33750210/145493511-92a40884-fe47-4588-9c60-2d5a4ed52452.png)

**Create repository**

<img src="https://user-images.githubusercontent.com/33750210/145493562-f9a8a2c6-a510-4a4e-b9da-9bb77d6df85a.png" alt="image" style="zoom:50%;" />

Repository name 작성 후 **Create**

![image](https://user-images.githubusercontent.com/33750210/145495949-43cb9de4-cca0-4f47-87a2-65a5b6b2a510.png)

SSH 선택해서 ssh url 복사

```shell
git clone <복사한 CodeCommit ssh url>
vim hello.txt # hello 작성
git add .
git commit -m "msg"
git push -u origin master
```

<img src="https://user-images.githubusercontent.com/33750210/145496228-f64b51ac-6443-460c-8d89-51377ce18db4.png" alt="image" style="zoom:50%;" />

성공적으로 CodeCommit에 hello.txt가 생성된 것을 확인할 수 있다.
