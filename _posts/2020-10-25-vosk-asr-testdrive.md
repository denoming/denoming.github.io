---
title: "VOSK: Test Drive"
date: 2020-10-25
categories: [Engineering,ASR]
tags: [asr,vosk]
---

Test drive will performed inside *Ubuntu 18.04* docker container:
```bash
$ docker run \
--name ubuntu \
--interactive \
--tty \
--user root:root \
--network=bridge \
ubuntu:18.04 /bin/bash
```

First we need install [VOSK](https://alphacephei.com/vosk/) and all prerequisites for it.

General dependencies:
```bash
$ apt update
$ apt install -y build-essential git swig
$ apt install -y automake autoconf unzip wget sox gfortran libtool  subversion zlib1g-dev
$ apt install -y python python-setuptools python2.7 python3 
```

Install Kaldi:
```bash
$ mkdir -p <path>
$ cd <path>
$ git clone https://github.com/kaldi-asr/kaldi.git kaldi --origin upstream
$ cd kaldi/tools
$ extras/install_mkl.sh
$ extras/install_openblas.sh
$ extras/check_dependencies.sh
extras/check_dependencies.sh: all OK.
$ make
$ cd ../src
$ ./configure --mathlib=OPENBLAS --shared --use-cuda=no
$ make -j$(nproc)
```

Install VOSK-API:
```bash
$ export KALDI_ROOT=<path>/kaldi # Specify where Kaldi located
$ git clone https://github.com/alphacep/vosk-api.git
$ cd vosk-api/python
$ python3 setup.py install --user --single-version-externally-managed --root=/
```

Test drive:
```bash
$ cd vosk-api/example
$ wget https://alphacephei.com/vosk/models/vosk-model-en-us-daanzu-20200905.zip
$ unzip vosk-model-en-us-daanzu-20200905.zip
$ mv vosk-model-en-us-daanzu-20200905 model
$ python3 ./test_simple.py test.wav
```

Note: Used as model - *Accurate wide-band model for dictation from Kaldi-active-grammar project*.

Useful Links:

1. [Models](https://alphacephei.com/vosk/models)
2. [Install instruction](https://alphacephei.com/vosk/install)