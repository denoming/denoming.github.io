---
title: "Git cheat sheet"
date: 2020-04-07
categories: [Development,CVS]
tags: [git]
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
$ git config --global user.email "NAME@example.com"
```

Configure username/email (repository):
```bash
$ git config user.name "FirstName LastName"
$ git config user.email "NAME@example.com"
$ git commit --amend --reset-author --no-edit # to reset author for previous commit
```

# Update

Update fork:
```git
$ git remote add upstream https://github.com/original-repo/goes-here.git # add additional remote
$ git fetch upstream # fetch all branches of remote upstream
$ git rebase upstream/master # rewrite master with upstream's master
$ git push origin master # push updates to master (may need --force)
```

# Aliases

The list of helpful git aliases: [link](https://github.com/karz0n/warehouse/blob/master/cfg/.gitconfig)

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

