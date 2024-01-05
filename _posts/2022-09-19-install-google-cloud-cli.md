---
title: "Install Google Cloud CLI"
date: 2022-09-19
categories: [Engineering,Linux]
tags: [google,cloud]
---

# Installing

```shell
$ sudo apt-get install apt-transport-https ca-certificates gnupg jq
$ echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
$ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
$ sudo apt-get update && sudo apt-get install google-cloud-cli
```

# Using TTS  service
Information about Google TTS service API is available [here](https://cloud.google.com/text-to-speech/docs/reference/rpc/google.cloud.texttospeech.v1).

Prepare request:
```shell
$ tee request.json > /dev/null <<EOF
{
  "input":{
    "text":"Android is a mobile operating system developed by Google, based on the Linux kernel and designed primarily for touchscreen mobile devices such as smartphones and tablets."
  },
  "voice":{
    "languageCode":"en-gb",
    "name":"en-GB-Standard-A",
    "ssmlGender":"FEMALE"
  },
  "audioConfig":{
    "audioEncoding":"LINEAR16"
  }
}
EOF
```

Synthesize text using Google TTS service:
```shell
$ export GOOGLE_APPLICATION_CREDENTIALS="$HOME/<path-to-json-key>.json"
$ curl -o response.json -X POST \
-H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
-H "Content-Type: application/json; charset=utf-8" \
-d @request.json \
"https://texttospeech.googleapis.com/v1/text:synthesize"
$ jq -r '.audioContent' response.json > audio.base64.txt
$ base64 audio.base64.txt -d > audio.wav
```
Synthesized text is avaible at `audio.wav` file.

Play synthesized text using gstreamer:
```shell
$ gst-launch-1.0 filesrc location=audio.wav ! wavparse ! audioconvert ! alsasink
```