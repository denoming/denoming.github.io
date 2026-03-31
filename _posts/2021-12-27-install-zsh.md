	---
title: "Install zsh"
date: 2021-12-27
categories: [Engineering,Linux]
tags: [zsh,tilix]
---

# Install zsh

```bash
$ sudo apt install zsh git
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

## Install nerd fonts

```bash
$ cd ~/Downloads
$ wget https://github.com/ryanoasis/nerd-fonts/releases/download/v2.0.0/Hack.zip
$ mkdir ~/.fonts
$ unzip Hack.zip -d ~/.fonts
# update fonts in terminal
```

# Install fonts

```shell
$ mkdir -p ~/.local/share/fonts/{CascadiaCode,CascadiaMono}
$ unzip ~/Downloads/CascadiaCode.zip -d ~/.local/share/fonts/CascadiaCode
$ unzip ~/Downloads/CascadiaMono.zip -d ~/.local/share/fonts/CascadiaMono
$ fc-cache -f -v
```

# Install shell emulators

## Install kitty emulator

Install kitty and modify theme:
```shell
$ sudo apt install kitty
$ kitten themes
```

Install shell prompt:
```shell
$ curl -sS https://starship.rs/install.sh | sh
$ vim ~/.zshrc
eval "$(starship init zsh)"
$ starship preset gruvbox-rainbow -o ~/.config/starship.toml
```
