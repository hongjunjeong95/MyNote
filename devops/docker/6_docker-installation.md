# docker installation

docker는 win이나 mac보다 linux에서 사용하는 것이 가장 바람직하다. 특히나 우분투 경우 가장 많은 학습자료가 모아져 있다. 그러므로 aws ec2 instance를 켜서 docker를 설치하자.

#### 권장 ec2 사양

**t3.small**을 사용하자. docker와 compose는 t2.small로 충분하지만 추후에 사용할 kubernetes의 minikube를 사용하는데 이에 필요한 사양이 2core cpu와 2GB RAM이다.

## Docker(Ubuntu)

```sh
#!/usr/bin/env bash
## INFO: https://docs.docker.com/engine/install/ubuntu/

set -euf -o pipefail

DOCKER_USER=ubuntu

# Install dependencies
sudo apt-get update && sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release

# Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --yes --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up the stable repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker CE
sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# Use Docker without root
sudo usermod -aG docker $DOCKER_USER
```

## Docker-compose(Ubuntu)

```sh
#!/usr/bin/env bash
## INFO: https://docs.docker.com/compose/install/

set -euf -o pipefail

DOCKER_COMPOSE_VERSION=2.1.1

# Download and install
sudo curl -L "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Docker(Mac)

```shell
brew install --cask docker
```
