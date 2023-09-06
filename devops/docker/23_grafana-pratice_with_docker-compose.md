# Docker - Grafana Practice

**Ref**

* [Grafana 도커 가이드](https://grafana.com/docs/grafana/latest/installation/docker/)
* [MySQL 도커 가이드](https://hub.docker.com/_/mysql)

## docker-compose.yml

```yaml
version: '3.9'

services:
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: grafana
      MYSQL_DATABASE: grafana
      MYSQL_USER: grafana
      MYSQL_PASSWORD: grafana
    volumes:
    - mysql-data:/var/lib/mysql
    logging:
      driver: "json-file"
      options:
        max-size: "8m"
        max-file: "10"

  grafana:
    depends_on:
    - db
    image: grafana/grafana:8.2.2
    restart: unless-stopped
    environment:
      GF_INSTALL_PLUGINS: grafana-clock-panel
    ports:
    - 3000:3000
    volumes:
    - ./files/grafana.ini:/etc/grafana/grafana.ini:ro
    - grafana-data:/var/lib/grafana
    logging:
      driver: "json-file"
      options:
        max-size: "8m"
        max-file: "10"

volumes:
  mysql-data: {}
  grafana-data: {}
```

* my-sql
  * `mysql-data:/var/lib/mysql` : 컨테이너 db에 데이터를 mysql-data 볼륨에 저장
* grafana
  * `./files/grafana.ini:/etc/grafana/grafana.ini:ro` : grafana.ini 설정 파일을 `:ro`인 읽기전용으로 볼륨 설정
  * `grafana-data:/var/lib/grafana` : grafana 데이터를 grafana-data 볼륨에 저장

## files/grafana.ini

```
app_mode = production
instance_name = ${HOSTNAME}

#################################### Server ####################################
[server]
protocol = http
http_addr =
http_port = 3000

#################################### Database ####################################
[database]
type = mysql
host = db:3306
name = grafana
user = grafana
password = grafana


#################################### Logging ##########################
[log]
mode = console
level = info

#################################### Alerting ############################
[alerting]
enabled = true
```

`.ini`파일을 읽고 grafana는 자신을 설정한다.
