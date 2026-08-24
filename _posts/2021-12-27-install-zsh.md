---
title: "Install zsh"
date: 2021-12-27
categories: [Engineering,Linux]
tags: [zsh,tilix]
---

# Install zsh

```bash
$ sudo apt install zsh git curl
$ curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh; zsh
$ sudo usermod --shell $(which zsh) $(whoami)
# restart shell
```

# Install plugins

## Install zsh-syntax-highliting

```bash
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
$ vim ~/.zshrc
...
plugins=(zsh-syntax-highlighting)
...
$ source ~/.zshrc
```

## Install zsh-autosuggestions

```bash
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
$ vim ~/.zshrc
...
plugins=(zsh-autosuggestions)
...
$ source ~/.zshrc
```
# Install fonts

```shell
$ wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.4.0/DejaVuSansMono.zip
$ wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.4.0/3270.zip
$ mkdir -p ~/.local/share/fonts/{NerdFont-DejaVuSansMono,NerdFont-3270}
$ unzip  ~/Downloads/DejaVuSansMono.zip -d ~/.local/share/fonts/NerdFont-DejaVuSansMono
$ unzip  ~/Downloads/3270.zip -d ~/.local/share/fonts/NerdFont-3270
$ fc-cache -f -v
```

# Install shell prompt:

```shell
$ curl -sS https://starship.rs/install.sh | sh
```

Add the following to the end of `~/.zshrc`:
```shell
$ vim ~/.zshrc
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8
...
eval "$(starship init zsh)"
```

Configure preset:
```shell
$ starship preset gruvbox-rainbow -o ~/.config/starship.toml
```
Note: Install particular required fonts and set in shell emulator.

# Install shell emulators

## Install kitty emulator

Install kitty and modify theme:
```shell
$ sudo apt install kitty
$ kitten themes
```




