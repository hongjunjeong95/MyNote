# SQL 데이터 덤프 방법

## PostgresSQL 데이터 덤프 방법

```sh
# 데이터베이스 덤프
$ pg_dump -h <host> -U <username> -d <db_name> -W -f <name>.sql

# 덤프 파일 실행 방법1
psql \
-f mydb2dump.sql \
--host localhost \
--port 5434 \
--username myawsuser \
--password password \
--dbname mydb

# 덤프 파일 실행 방법2
pg_restore -h localhost -p 5432 -U <username> -d <db_name> ../<dump_file_name>.dump
```

### 실행 예제

```sh
psql \
-f data.sql \
-h localhost \
-p 5434 \
-U postgres \
-d datahunt
```



## MySQL 데이터 덤프 방법

```sh
# 데이터베이스 덤프
$ mysqldump -h <host> -u <username> --databases moyamo_db -p > <name>.sql

# scp로 원격 서버에서 로컬 서버로 덤프 파일 전송(덤프 파일이 원격 서버에 있을 경우에만)
$ scp <user>@<원격 IP>:<원격 파일 경로> <저장될 로컬 파일 경로>

# mysql 접속
$ mysql \
-h 127.0.0.1 \
-P 3306 \
--user=root \
--password

# mysql에서 덤프 파일 실행
mysql> source <.sql파일의 path>

또는

$ mysql -h <host> -u <user> <db_name> < <.sql 파일의 path>
```

### 실행 예제

```sh
$ mysql \
-h 127.0.0.1 \
-P 3306 \
--user=root \
-p

mysql> source ~/test.sql


mysqldump -h moyamo-product-backup-20220907-cluster.cluster-c7mmfvbcxzve.ap-northeast-2.rds.amazonaws.com -u moyamodb -p --all-databases > moyamo_all_db.sql
```

