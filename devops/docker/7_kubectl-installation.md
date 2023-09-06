# kubectl installation

docker는 win이나 mac보다 linux에서 사용하는 것이 가장 바람직하다. 특히나 우분투 경우 가장 많은 학습자료가 모아져 있다. 그러므로 aws ec2 instance를 켜서 docker를 설치하자.

#### 권장 ec2 사양

**t3.small**을 사용하자. docker와 compose는 t2.small로 충분하지만 추후에 사용할 kubernetes의 minikube를 사용하는데 이에 필요한 사양이 2core cpu와 2GB RAM이다.

## kubectl(Ubuntu)

```sh
#!/usr/bin/env bash
## INFO: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management

set -euf -o pipefail

# Install dependencies
sudo apt-get update && sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release

# Add kubectl's official GPG key
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg

# Set up the repository
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install kubectl
sudo apt-get update && sudo apt-get install -y kubectl
```

## kustomize(Ubuntu)

```sh
#!/usr/bin/env bash
## INFO: https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/

set -euf -o pipefail

KUSTOMIZE_VERSION=v4.4.1

# Download kustomize binary
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/kustomize/${KUSTOMIZE_VERSION}/hack/install_kustomize.sh"  | bash

# Install to /usr/local/bin
sudo install -o root -g root -m 0755 kustomize /usr/local/bin/kustomize
```

## Mac - kubectl & kustomize

```sh
brew install kubectl
brew install kustomize
```

도커 자체에 `kubectl`이 내장된 버전이 있다. `kubectl`을 설치하다가 기존의 도커와 충돌이 날 경우 아래 명령어로 덮어씌울 수 있다.

```shell
brew link --overwrite kubernetes-cli
```

