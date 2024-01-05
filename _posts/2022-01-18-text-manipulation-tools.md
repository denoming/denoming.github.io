---
title: "Text manipulation tools"
date: 2021-12-27
categories: [Engineering,Linux]
tags: [regexp]
---

# AWK

Prints first column:
```sh
$ ps | awk '{print $1}'

```
Prints first column with specifying custom column delimitier:
```sh
$ awk -F ":" '{print $1}' /etc/passwd
```
Prints the last field with filtering:
```sh
$ awk -F "/" '/^\// {print $NF}' /etc/shells | uniq
```
or
```sh
$ df -h | awk '/\/dev\/sd/ {print $1"\t"$5}'

```
or
```sh
$ ps -ef | awk '$1 ~ /(root|colord)/ {print $1"\t"$2}'
```
Prints using if clause:
```sh
$ ps -ef | awk '{ if($NF == "/usr/bin/zsh") print $0 }'
```
Prints lines with truncating:
```sh
# 01 Twm
# 02 PekWM
# 03 JVM
$ awk '{print substr ($0, 4)}' file.txt
# Twm
# PekWM
# JVM
```
Prints with matching:
```sh
$ awk 'match($0,/lib/) {print $1}' /etc/group
```
Prints specific number of records:
```sh
$ df | awk 'NR==3, NR==5 {print $0}'
```

# SED

Simple replaces:
```sh
$ echo Hello | sed s/Hello/Goodbye/
Goodbye
```

# GREP

*  grep - grep tool that uses BRE (basic regular expression)
* egrep - grep tool that uses ERE (extented regilar expression)

Searches given pattern `class` in the files that match given glob pattern:
* `grep . --include="*.hpp" -rnw -e "class"`

## Perl

Finds lines in given file that contain article:
```sh
$ perl -ne 'print if /the|The|THE/' file.txt
$ perl -ne 'print if /(?i)the/' file.txt
$ perl -ne 'print if /the/i' file.txt
```
Finds given pattern and substitute with changing order:
```sh
$ perl -ne 'print if s/(Some string) (with grouping)/$2 $1/' file.txt
```

# ACK

```sh
#          posititve lookahead
# (matches all what followed by 'ma' pattern)
$ ack -Hi 'ancyent (?=ma)' rime.txt
#         negative lookahead
# (matches all what not followed by 'marinere' pattern)
$ ack -Hi 'ancyent (?!marinere)' rime.txt
#         positive lookbehind 
# (matches all what followed after 'ancyent' pattern)
$ ack -Hi '(?<=ancyent) marinere'
#         negative lookbehind
# (matches all what not followed after 'ancyent' pattern)
$ ack -Hi '(?<!ancyent) marinere'
```

# Templates

E-mail regular expression:
`^([\w\-.!#$%&;'*+-\/=?^_'{|}~]+)@(\w+)\.([a-zA-Z]{2,4})$`