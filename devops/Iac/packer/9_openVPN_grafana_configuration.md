# Packer - openVPN and Grafana Configuration

**Ref**

* [OpenVPN](https://github.com/wheelybird/openvpn-server-ldap-otp)
* [tunnelblick](https://tunnelblick.net/) : Mac OS OpenVPN Client

## openvpn-userdata.sh

```sh
#!/bin/bash
sudo apt-get update
sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
usermod -aG docker ubuntu

## Run openvpn-ldap-otp container
docker run \
 --name openvpn \
 --volume openvpn-data:/etc/openvpn \
 --detach=true \
 -p 1194:1194/udp \
 --cap-add=NET_ADMIN \
 -e "OVPN_SERVER_CN=${public_ip}" \
 -e "OVPN_ENABLE_COMPRESSION=false" \
 -e "OVPN_NETWORK=172.22.16.0 255.255.240.0" \
 -e "OVPN_ROUTES=172.22.16.0 255.255.240.0, ${split("/", vpc_cidr)[0]} ${cidrnetmask(vpc_cidr)}" \
 -e "OVPN_NAT=true" \
 -e "OVPN_DNS_SERVERS=${cidrhost(vpc_cidr, 2)}" \
 -e "USE_CLIENT_CERTIFICATE=true" \
 wheelybird/openvpn-ldap-otp:v1.4

## Wait to ready OpenVPN Server
until echo "$(docker exec openvpn show-client-config)" | grep -q "END PRIVATE KEY" ;
do
  sleep 1
  echo "working..."
done

## Generate OpenVPN client configuration file
docker exec openvpn show-client-config > fastcampus.ovpn
```

## 실습 순서

1. 인프라 구성

   1. 2-terraform/lab-openvpn/network 실행
   2. 2-terraform/lab-openvpn/ec2-instances 실행

2. openvpen instance 접속

3. sudo su -

4.  `cat /var/log/cloud-init-output.log`를 통해서 도커가 이미지를 잘 가져왔는지 확인 가능

5. cat fastcampus.ovpn

   1. 해당 값 복사

6. local pc로 오기

7. ````shell
   cat > fastcampus.ovpn
   fjeiwngklsrqoiweu128937128
   -----END PRIVATE KEY-----
   CTRL + D
   ````

8. tunnelblick 설치

   ```shell
   brew install tunnelblick
   ```

9. fastcampus.ovpn 실행

10. Mac 상단 바에서 tunnelblick 아이콘 클릭

11. **Connect fastcampus** 클릭

12. Connected 확인

13. openvpn private ip로 ssh 접속

14. nslookup 명령어로 private dns에 dns 질의를 날려서 connection을 확인
