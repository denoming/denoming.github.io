Install sigc++-3.0:
```shell
$ sudo apt install mm-common docbook-xsl
$ git clone -b 3.2.0 git@github.com:libsigcplusplus/libsigcplusplus.git
$ cd libsigcplusplus
$ git switch -c 3.2.0
$ meson --prefix=$HOME/.local --libdir=lib bld .
$ cd bld
$ ninja
$ ninja install
$ ninja test
```

Install glibmm 2.72.1:
```shell
$ sudo apt install mm-common docbook-xsl
$ git clone -b 2.72.1 git@github.com:GNOME/glibmm.git
$ cd glibmm
$ git switch -c 2.72.1
$ meson --prefix=$HOME/.local --libdir=lib bld .
$ cd bld
$ ninja
$ ninja install
$ ninja test
```
