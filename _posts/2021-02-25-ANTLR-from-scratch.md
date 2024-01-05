---
title: "ATNLR from scratch"
date: 2021-02-25
categories: [Language Recognition]
tags: [antlr,ll]
---

# Install

## Install backend

```bash
$ wget https://www.antlr.org/download/antlr-4.9.1-complete.jar
$ sudo cp antlr-4.9.1-complete.jar /usr/local/lib
$ tee -a ~/.bashrc > /dev/null <<'EOF'

# Configure ATNLR
export CLASSPATH=".:/usr/local/lib/antlr-4.9.1-complete.jar:$CLASSPATH"
alias antlr4='java -Xmx1G -cp "/usr/local/lib/antlr-4.9.1-complete.jar:$CLASSPATH" org.antlr.v4.Tool'
alias grun='java org.antlr.v4.gui.TestRig'
EOF
$ source ~/bashrc
```

## Install Runtime

```bash
$ sudo apt install uuid-dev
$ wget https://www.antlr.org/download/antlr4-cpp-runtime-4.9.1-source.zip
$ unzip antlr4-cpp-runtime-4.9.1-source.zip -d antlr4-cpp-runtime-4.9.1-source
$ cd antlr4-cpp-runtime-4.9.1-source
$ mkdir -p {build,run} && cd build
$ cmake -DANTLR4_INSTALL=ON -DCMAKE_INSTALL_PREFIX=${HOME}/opt/antlr4 ..
$ make -j$(nproc)
$ make install
```

# Generate

```bash
$ antlr4 -Dlanguage=Cpp -no-listener -visitor -o <output> <name-of-grammar>.g4
```

