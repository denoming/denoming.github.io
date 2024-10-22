---
title: "Using GPG tool"
date: 2024-10-21
categories: [Engineering,Linux]
tags: [gpg]
---

# Generating

## Generating in batch mode

Generate RSA key with 4096 len and save at `GNUPGHOME` dir:
```shell
$ export GNUPGHOME="$(mktemp -d $HOME/pgpkeys-XXXXXX)"
$ gpg --no-tty --batch --gen-key <<EOF
%echo Generating PGP key
Key-Type: RSA
Key-Length: 4096
Name-Real: Maintainer
Name-Email: maintainer@denoming.com
Expire-Date: 0
%no-ask-passphrase
%no-protection
%commit
EOF
```

Generate EdDSA key and save at `GNUPGHOME` dir:
```shell
$ export GNUPGHOME="$(mktemp -d $HOME/pgpkeys-XXXXXX)"
$ gpg --expert --no-tty --batch --gen-key <<EOF
%echo "Generating ECC keys (sign & encrypting) with no-expiry"
Key-Type: EDDSA
Key-Curve: ed25519
Subkey-Type: ECDH
Subkey-Curve: cv25519
Name-Real: Denys Asauliak
Name-Email: d.asauliak@gmail.com
Expire-Date: 0
%no-ask-passphrase
%no-protection
%commit
EOF
```

# Import and export

List keys:
```shell
$ gpg --list-keys
...
pub   rsa4096 2024-10-20 [SCEA]
      3CE69396BD6BCC5CB58C3E55C579AEA5D7E2BA6F
uid           [ultimate] <key-name> <example@example.com>
```

Export:
```shell
# Public key
$ gpg --armor --export "<key-name>" > $HOME/<key-name>.public
# Private
$ gpg --armor --export-secret-keys "<key-name>" > $HOME/<key-name>.private
```

Import:
```shell
$ cat example.(public|private) | gpg --import
```

# Signing

```shell
$ cat "<some-file>" | gpg --default-key "<key-name>" -abs > "<some-file>.gpg"
```

# Misc

Get key's ID:
```shell
$ gpg --keyid-format short --list-keys | grep "pub " | awk -F "/" "{ print $2 }" | awk -F " " "{ print $1 }"
pub   <algo>/<key-ud> date ...
```

# Links

* [Unattended key generation (batch mode)](https://www.gnupg.org/documentation/manuals/gnupg-devel/Unattended-GPG-key-generation.html)