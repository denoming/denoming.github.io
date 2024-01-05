---
title: "CLI cheatsheet"
date: 2021-12-27
categories: [Engineering,Linux]
tags: [cli,tilix]
---

# CLI tips and tricks

## Save current directory to the stack

```sh
$ pushd /etc
$ cd /var
$ popd
```

## Send program to background

```sh
$ vim ~/.bashrc
<Press Ctrl+Z>
$ fg
```

## Paste the most recent command

```sh
$ apt update
$ sudo !!
```

## Find command in the history

- To find the commnad among the history type: `Ctrl+R`

- To display all history: `$ history`

- To run the command from the history: `$ !<number>`

- To display time next to each command in the history:
  
  ```
  $ HISTTIMEFORMAT="%Y-%m-%d %T"
  $ history
  ```
  
  Setup it permanently:
  
  ```sh
  $ vim ~/.bashrc
  HISTTIMEFORMAT="%Y-%m-%d %T"
  ```
  
  or
  
  ```sh
  $ vim ~/.zshrc
  HIST_STAMPS="%d.%m.%y %H:%M:%S"
  ```

## Truncate file

```sh
$ truncate -s 0 file.txt
```

## Columnize output

```sh
$ mount | column -t
```