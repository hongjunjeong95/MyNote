# Issues

## Add

------

### git add 취소하기

- git reset HEAD <file> : 파일을 unstaged 할 수 있다.
- git reset HEAD : add한 파일 전체를 취소할 수 있다.

## Clone

------

### git clone 예전 버전 가져오기

1. git clone 깃 저장 주소
2. cd 프로젝트명
3. git reset —hard 커밋해시

### git commit amend remote repository

1. 파일 수정
2. git add .
3. git commit —amend
4. git push -f



## Config

------

> warning: LF will be replaced by CRLF in file The file will have its original line endings in your working directory

1. Window는 파일들을 CRLF 형식으로 git에 올리고 POSIX 기반 운영체제는 LF로 올린다.
2. 나는 eslint가 LF를 요구하므로 LF 형식으로 올리겠다. 그러기 위해서 core.autocrlf를 false로 한다.
3. git config —global core.autocrlf false



## gitignore

------

### gitignore 적용 안 될 때

1. git rm -r —cached .
2. git add .
3. git commit -m "~~~"



## ec2 git pull permission error

------

> error: cannot open .git/FETCH_HEAD: Permission denied

```sh
$ sudo chown -R $User .git/
```

