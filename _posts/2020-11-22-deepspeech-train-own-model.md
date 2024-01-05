---
title: "DeepSpeech: Train own model"
date: 2020-10-25
categories: [Engineering,ASR]
tags: [asr,vosk]
---

## Prepare

### Prepare datasets

Create directories:

```bash
$ mkdir -p ~/temp/asr/{train,dev,test,export,tools}
$ cd ~/temp/asr
$ export ASR=$PWD
```

Purpose of each directory:

- **train** - for train dataset;
- **dev** - for dev dataset;
- **test** - for test dataset;
- **export** - for outcome artifacts (model, scorer, vocabulary, checkpoints);
- **tools** - for deep speech source code and native client.

We will teach our model to recognize some input spelled by our voice. Our input data is consist of four sentences:

```
turn on the light
turn off the light
switch on the light
switch off the light
```

So we create text file for this four sentences:

```bash
$ tee $ASR/input.txt <<EOF
> turn on the light
> turn off the light
> switch on the light
> switch off the light
> EOF
```

Next we should record these sentences, spelled by our voice with different speed, intonation and so on. The more records you make then better result you will get. Our datasets should contain records for each meaningful word. To record I recommend to use [Audacity](https://www.audacityteam.org/) tool.

Datasets of different type (train, dev, test) has to contain records in proportion: **70% for train**, **20% for dev** and **10% for test**. 

All records should be placed in particular directory. Each dataset directory should contains meta file that list content of that directory:

```bash
$ tee $ASR/train.csv <<EOF
wav_filename,wav_filesize,transcript
/asr/train/train-01.wav,62640,turn on the light
...
EOF
$ tee $ASR/dev/dev.csv <<EOF
wav_filename,wav_filesize,transcript
/asr/dev/dev-01.wav,36200,turn on
...
$ tee $ASR/test/test.csv <<EOF
wav_filename,wav_filesize,transcript
/asr/test/test-01.wav,58470,turn on the light
...
```

As you can see path for each record should start from */asr* root. Later, we will see why.

As a result we should obtain structure of directories like this:

```bash
$ tree $ASR
.
├── dev
│   ├── dev-01.wav
│   │   ...
│   └── dev.csv
├── export
├── input.txt
├── test
│   ├── test-01.wav
│   ├── ...
│   └── test.csv
└── train
    ├── train-01.wav
    ├── ...
    └── train.csv
```

### Prepare train environment

Build train docker image:

```bash
$ cd $ASR/tools
$ git clone --branch v0.9.1 https://github.com/mozilla/DeepSpeech && cd DeepSpeech
$ make Dockerfile.train DEEPSPEECH_REPO=https://github.com/mozilla/DeepSpeech.git DEEPSPEECH_SHA=v0.9.1
$ cat Dockerfile.train | docker build -t asr/deepspeech:train -
```

Install deepspeech native client:

```bash
$ cd $ASR/tools
$ wget https://github.com/mozilla/DeepSpeech/releases/download/v0.9.1/native_client.amd64.cpu.linux.tar.xz
$ mkdir native-client 
$ tar xf native_client.amd64.cpu.linux.tar.xz -C native-client 
$ rm native_client.amd64.cpu.linux.tar.xz
```

## Generate

### Generate language model

```bash
$ docker run -it --rm \
-u root:root \
-v ${ASR}:/asr \
asr/deepspeech:train /bin/bash
$ cd data/lm
$ python3 generate_lm.py --input_txt /asr/input.txt --output_dir /asr/export \
  --top_k 500000 --kenlm_bins /DeepSpeech/native_client/kenlm/build/bin/ \
  --arpa_order 5 --max_arpa_memory "85%" --arpa_prune "0|0|1" \
  --binary_a_bits 255 --binary_q_bits 8 --binary_type trie --discount_fallback
...
Building lm.binary ...
...
SUCCESS
$ ls /asr/export
lm.binary  vocab-500000.txt
$ exit
```

For more information follow this [link](https://deepspeech.readthedocs.io/en/v0.9.1/Scorer.html).

### Generate scorer

```bash
$ cd $ASR/tools/native-client
$ ./generate_scorer_package \
--alphabet $ASR/tools/DeepSpeech/data/alphabet.txt \
--lm $ASR/export/lm.binary \
--vocab $ASR/export/vocab-500000.txt \
--package $ASR/export/kenlm.scorer \
--default_alpha 0.931289039105002 \
--default_beta 1.1834137581510284
```
Values for *alpha* and *beta* by default: 0.931289039105002 and 1.1834137581510284 respectively. In [Building your own scorer](https://deepspeech.readthedocs.io/en/v0.9.1/Scorer.html#building-your-own-scorer) topic describing how to obtain more relevant values for them.

### Generate model

```bash
$ docker run -it --rm \
-u root:root \
-v ${ASR}:/asr \
asr/deepspeech:train /bin/bash
$ python3 -u DeepSpeech.py \
  --train_files /asr/train/train.csv \
  --dev_files /asr/dev/dev.csv \
  --test_files /asr/test/test.csv \
  --n_hidden 134 \
  --epochs 75 \
  --early_stop True \
  --export_dir /asr/export \
  --checkpoint_dir /asr/export/checkpoint \
  --scorer_path /asr/export/kenlm.scorer
I Could not find best validating checkpoint.
I Could not find most recent checkpoint.
I Initializing all variables.
I STARTING Optimization
Epoch 0 |   Training | Elapsed Time: 0:00:03 | Steps: 66 | Loss: 39.314777                                           
Epoch 0 | Validation | Elapsed Time: 0:00:01 | Steps: 22 | Loss: 31.439259 | Dataset: /asr/dev/dev.csv               
I Saved new best validating model with loss 31.439259 to: /asr/export/checkpoint/best_dev-66
...
Test on /asr/test/test.csv - WER: 0.125000, CER: 0.200000, loss: 18.478329
...
$ ls /asr/export
author_model_0.0.1.md  checkpoint  kenlm.scorer  lm.binary  output_graph.pb  vocab-500000.txt
$ convert_graphdef_memmapped_format --in_graph=/asr/export/output_graph.pb --out_graph=/asr/export/output_graph.pbmm
...Converted 6 nodes

Model 	: output_graph.pbmm
Scorer 	: kenlm.scorer
$ exit
```

Our final artifacts:

- *output_graph.pbmm* - model
- *kenlm.scorer* - scorer

Using previous installed native client we can use our final artifacts to recognize recorded sound:

```bash
$ cd $ASR/tools/native-client
$ deepspeech \
--model $ASR/export/output_graph.pbmm \
--scorer $ASR/export/kenlm.scorer \
--audio <input>.wav
...
```

