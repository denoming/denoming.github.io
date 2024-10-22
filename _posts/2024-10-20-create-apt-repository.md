---
title: "Create apt repository"
date: 2024-10-20
categories: [Engineering,Linux]
tags: [apt,deb,linux]
---
# Configure

## Manual configuration

Installing tools:
```shell
$ sudo apt install -y gcc dpkg-dev gpg
```

Create repository root dir:
```shell
$ export DIR=/data
$ mkdir -p $DIR/apt/pool/main
```
The location `/data` might be changed.

Create dirs for each architectures (e.g. `arm64`, `amd64`, etc):
```shell
$ mkdir -p $DIR/apt/dists/stable/main/binary-arm64
...
```

Copy `*deps` packages to main dir:
```shell
$ cp *.deb $DIR/apt/pool/main
```

Generate `Packages` file (`arm64` architecture):
```shel
$ cd $DIR/apt
$ dpkg-scanpackages --arch arm64 pool > dists/stable/main/binary-arm64/Packages
$ cat dists/stable/main/binary-arm64/Packages | gzip -9 > dists/stable/main/binary-arm64/Packages.gz
```

Generate `Release` file (`arm64` architecture):
```shell
$ cd $DIR/apt/dists/stable
$ generate-release.sh > Release
```
### Signing

Create GPG key:
```shell
$ export GNUPGHOME="$(mktemp -d $HOME/pgpkeys-XXXXXX)"
$ gpg --no-tty --batch --gen-key <<EOF
%echo Generating PGP key
Key-Type: RSA
Key-Length: 4096
Name-Real: Maintainer
Name-Email: maintainer@example.com
Expire-Date: 0
%no-ask-passphrase
%no-protection
%commit
EOF
$ gpg --list-keys
...
pub   rsa4096 2024-10-20 [SCEA]
      3CE69396BD6BCC5CB58C3E55C579AEA5D7E2BA6F
uid           [ultimate] Maintainer <maintainer@example.com>
```

Export public and private keys for further backup:
```shell
# Export public key
$ gpg --armor --export "Maintainer" > $HOME/maintainer.public
$ cat $HOME/maintainer.denoming.public | gpg --list-packets
# Export private key
$ gpg --armor --export-secret-keys "Maintainer" > $HOME/maintainer.denoming.private
```

Copy public key to the repository root
```shell
$ cp $HOME/maintainer.public $DIR/apt/maintainer.gpg
```

Signing (creating `Release.gpg` and `InRelease` files):
```shell
$ cat $DIR/apt/dists/stable/Release | gpg --default-key "Maintainer" -abs > $DIR/apt/dists/stable/Release.gpg
$ cat $DIR/apt/dists/stable/Release | gpg --default-key "Maintainer" -abs --clearsign > $DIR/apt/dists/stable/InRelease
```
### Run

```shell
$ cd $DIR/apt
$ python3 -m http.server 8080
```
# Using

Saving public GPG key:
```shell
$ sudo mkdir -m 0755 -p /etc/apt/keyrings
$ curl -fsSL http://<hostname>:8080/maintainer.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/denoming.gpg
```

Add source list:
```shell
$ echo "deb [arch=arm64 signed-by=/etc/apt/keyrings/denoming.gpg] http://<hostname>:8080/apt stable main" | sudo tee /etc/apt/sources.list.d/<hostname>.list > /dev/null
```

Gather update:
```shell
$ sudo apt update
```
# Scripts

```shell
$ sudo tee /usr/local/bin/generate-release.sh > /dev/null <<'END'
#!/usr/bin/env bash
set -e

do_hash() {
    HASH_NAME=$1
    HASH_CMD=$2
    echo "${HASH_NAME}:"
    for f in $(find -type f); do
        f=$(echo $f | cut -c3-) # remove ./ prefix
        if [ "$f" = "Release" ]; then
            continue
        fi
        echo " $(${HASH_CMD} ${f}  | cut -d" " -f1) $(wc -c $f)"
    done
}

cat << EOF
Origin: DENOMING Repository
Label: DENOMING
Suite: stable
Codename: stable
Version: 1.0
Architectures: arm64
Components: main
Description: DENOMING software repository
Date: $(date -Ru)
EOF
do_hash "MD5Sum" "md5sum"
do_hash "SHA1" "sha1sum"
do_hash "SHA256" "sha256sum"
END
$ sudo chmod +x /usr/local/bin/generate-release.sh
```