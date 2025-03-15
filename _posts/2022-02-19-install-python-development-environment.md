
# Configuring

Installing `venv` package:
```shell
$ python3 -m pip install --user --upgrade pip
$ python3 -m pip install --user virtualenv
```

## Per-project environment

```shell
$ python3 -m venv .venv
$ source .venv/bin/activate
$ pip3 install -r requirements.txt
$ deactivate
```

## Per-user environment

```shell
$ python3 -m venv ~/.venv
$ source ~/.venv/bin/activate
$ pip3 install -r requirements.txt
$ deactivate
```