# Packer - Installation

## Installation

**Ref**

* https://learn.hashicorp.com/tutorials/packer/get-started-install-cli

### Installation

First, install the HashiCorp tap, a repository of all our Homebrew packages.

```shell
brew tap hashicorp/tap
```

Now, install Packer with `hashicorp/tap/packer`.

```shell
brew install hashicorp/tap/packer
packer version
```

### Enable tab completion

```shell
touch ~/.zshrc
```

Then install the autocomplete package.

```shell
packer -autocomplete-install
```

It shoud exist on `~/.zshrc` file

<img src="https://user-images.githubusercontent.com/92770273/144362386-58a56ab0-4079-4965-9709-71dace4ae020.png" alt="image" style="zoom:50%;" />

Once the autocomplete support is installed, you will need to restart your shell.

```shell
zsh
```

you can auto complete packer command `packer + tab`
