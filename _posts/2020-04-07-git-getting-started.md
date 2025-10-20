---
title: Git - Getting started
date: 2020-04-07
categories:
  - Development
  - CVS
tags:
  - git
---
# Install

```shell
$ sudo apt install git git-lfs git-gui
```
# Configure

Generate SSH key:
```
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
> ...
$ eval "$(ssh-agent -s)"
> Agent pid ....
$ ssh-add ~/.ssh/id_rsa
...
```
Configure username/email (global):
```bash
$ git config --global user.name "FirstName LastName"
$ git config --global user.email "name@example.com"
```

Configure username/email (repository):
```bash
$ git config user.name "FirstName LastName"
$ git config user.email "name@example.com"
$ git commit --amend --reset-author --no-edit # to reset author for previous commit
```

The example of config to use: [link](https://github.com/denoming/warehouse/blob/d207458ccef2d136ac23fb2c1648695fe21a8a2a/cfg/.gitconfig)
# Update

Update fork:
```git
$ git remote add upstream https://github.com/original-repo/goes-here.git # add additional remote
$ git fetch upstream # fetch all branches of remote upstream
$ git rebase upstream/master # rewrite master with upstream's master
$ git push origin master # push updates to master (may need --force)
```
# Misc

The list of commonly used commit prefixes:
```text
fix:
feat: 
build:
chore:
ci:
docs:
refactor:
perf:
test:
```
# Links

* [git-config](https://git-scm.com/docs/git-config)
