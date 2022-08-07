# vTerraform - Installation

## Installation

**Ref**

* https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started

### Installation

First, install the HashiCorp tap, a repository of all our Homebrew packages.

```shell
brew tap hashicorp/tap
```

Now, install Terraform with `hashicorp/tap/terraform`.

```shell
brew install hashicorp/tap/terraform
terraform version
```

### Enable tab completion

```shell
touch ~/.zshrc
```

Then install the autocomplete package.

```shell
terraform -install-autocomplete
```

It shoud exist on `~/.zshrc` file

<img src="https://user-images.githubusercontent.com/92770273/144361148-4f008393-4a3b-4ab8-833b-d3034af0025f.png" alt="image" style="zoom:50%;" />

Once the autocomplete support is installed, you will need to restart your shell.

```shell
zsh
```

you can auto complete terraform command `terraform + tab`

### Terraform cache

**Ref**

* https://www.terraform.io/docs/cli/config/config-file.htmlhttps://www.terraform.io/docs/cli/config/config-file.html

Add `plugin_cache_dir   = "$HOME/.terraform.d/plugin-cache"` to `~/.terraformrc`

Create directory

```shell
mkdir -p ~/.terraform.d/plugin-cache
```

plugin_cache_dir 설정을 통해 테라폼의 캐시 저장 공간을 중앙집중식으로 효율적으로 관리할 수 있다. That is, it is possible to centerally manage the terraform providers and modules each workspaces requires.
