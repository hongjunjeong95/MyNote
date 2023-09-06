# kail을 이용하여 쿠버네티스 로그 분석

**Ref**

* [kail](https://github.com/boz/kail)

## kail 설치

MacOS

```
brew tap boz/repo
brew install boz/repo/kail
```

Ubuntu

```
sudo su
bash <( curl -sfL https://raw.githubusercontent.com/boz/kail/master/godownloader.sh) -b /usr/local/bin
```



## Combining Selectors

Ref : https://github.com/boz/kail#combining-selectors

### 같은 flag를 사용하게 되면 **"OR"**로 동작한다.

```sh
kail --rs workers --rs db
```

workers나 db replicaset에 대한 파드 로그를 본다.



### 다른 flag를 사용하게 되면 **"AND"**로 동작한다.

```sh
kail --svc frontend --deploy webapp
```

service가 frontend이고 deployment가 webapp인 로그를 보여준다.



## Other Flags

--since DURATION : 특정 시점 전부터 보여준다. 이 플래그를 제공하지 않으면 실시간으로 스트리밍을 받는다.
