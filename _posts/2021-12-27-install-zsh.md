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
$ mkdir ~/.fonts && cd ~/.fonts
# update fonts in terminal
```

## Install Powerlevel9k

```bash
$ git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
$ vim ~/.zshrc file
...
ZSH_THEME="powerlevel9k/powerlevel9k"

POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(root_indicator ssh dir dir_writable vcs)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status background_jobs)
POWERLEVEL9K_MODE="nerdfont-complete"
POWERLEVEL9K_COLOR_SCHEME='light'
...
```
