---
title: "DeepSpeech: Test Drive"
date: 2020-10-25
categories: [Engineering,ASR]
tags: [asr,deepspeech]
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

First we need install [DeepSpeech](https://deepspeech.readthedocs.io/en/v0.8.2/USING.html#installing-deepspeech-python-bindings).

General dependencies:
```bash
$ apt update
$ apt install -y wget python3 pytho3-pip
```

Install DeepSpeech:
```bash
$ pip3 install deepspeech
```

Test drive:
```bash
$ wget https://github.com/mozilla/DeepSpeech/releases/download/v0.8.2/deepspeech-0.8.2-models.pbmm
$ wget https://github.com/mozilla/DeepSpeech/releases/download/v0.8.2/deepspeech-0.8.2-models.scorer
$ wget wget https://github.com/mozilla/DeepSpeech/releases/download/v0.8.2/audio-0.8.2.tar.gz
$ tar xf audio-0.8.2.tar.gz
$ deepspeech \
--model deepspeech-0.8.2-models.pbmm \
--scorer deepspeech-0.8.2-models.scorer \
--audio audio/2830-3980-0043.wav
Loading model from file deepspeech-0.8.2-models.pbmm
TensorFlow: v2.2.0-24-g1c1b2b9
DeepSpeech: v0.8.2-0-g02e4c76
...
experience proves this
...
Inference took 0.729s for 1.975s audio file.
```

Useful Links:
1. [DeepSpeech Releases](https://github.com/mozilla/DeepSpeech/releases)
