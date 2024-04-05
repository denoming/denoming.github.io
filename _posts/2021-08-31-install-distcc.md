---
title: "Install distcc"
date: 2021-08-31
categories: [Development,Tools]
tags: [distcc,tools]
---

# Prepare

Create `distcc` service user:
```bash
$ sudo useradd -m -c "distcc service user" distcc
$ sudo passwd distcc
```

# Setup service

Compile and install `distcc`:
```bash
$ sudo apt -y install git
$ sudo apt -y install gcc make autoconf automake python python-dev python3-dev libiberty-dev libgnomeui-dev
$ git clone https://github.com/distcc/distcc.git && cd distcc
$ ./autogen.sh
$ ./configure --disable-Werror
$ make
$ sudo make install
```

Download cross toolchain:
```bash
$ sudo git clone --depth=1 https://github.com/raspberrypi/tools.git /usr/lib/raspberrypi-tools
$ sudo ln -s /usr/lib/raspberrypi-tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/bin /usr/local/bin/cross-tools
$ cd /usr/local/bin/cross-tools
$ ln -s arm-linux-gnueabihf-gcc gcc
$ ln -s arm-linux-gnueabihf-gcc cc
$ ln -s arm-linux-gnueabihf-c++ c++
$ ln -s arm-linux-gnueabihf-g++ g++
$ ln -s arm-linux-gnueabihf-cpp cpp
```

Register `distcc` service as systemctl service:
```bash
$ tee > /dev/null /etc/systemd/system/distcc@.service <<EOF
[Unit]
Description=Distcc on %I user instance
After=network.target

[Service]
Type=idle
ExecStart=/usr/local/bin/distccd --daemon --jobs <number_of_jobs> --allow <allowed_network> --verbose --user %I --log-stderr --no-detach
Environment="PATH=/usr/local/bin/cross-tools:$PATH"

[Install]
WantedBy=multi-user.target
EOF
```
Specify appropriate values for `<number_of_jobs>` and `<allowed_network>` placeholder.

Enable `distss` service:
```bash
$ sudo systemctl enable distcc@distcc
$ sudo systemctl start distcc@distcc
```

# Setup client

Install client:
```bash
$ sudo apt -y install git
$ sudo apt -y install gcc make autoconf automake python python-dev python3-dev libiberty-dev
$ git clone https://github.com/distcc/distcc.git distcc && cd distcc
$ ./autogen.sh
$ ./configure --disable-Werror
$ make
$ sudo make install
```

Configure bash:
```text
# The remote machines that will build things for you.
# The syntax is : "IP_ADDRESS/NUMBER_OF_JOBS"
# Formula: number of processors x 2
export DISTCC_HOSTS="<server_address>/<number_of_jobs>,lzo,cpp"

# When a job fails, distcc backs off the machine that failed for some time.
# We want distcc to retry immediately
export DISTCC_BACKOFF_PERIOD=0

# Time, in seconds, before distcc throws a DISTCC_IO_TIMEOUT error and tries to build the file
# locally (default hardcoded to 300 in version prior to 3.2)
export DISTCC_IO_TIMEOUT=3000

# Don't try to build the file locally when a remote job failed
export DISTCC_SKIP_LOCAL_RETRY=1
```

# Use

```bash
$ mkdir -p ~/workspace/test-project
$ tee > /dev/null ~/workspace/test-project/test.cpp <<EOF
int main()
{
    std::cout << "Hello world !!!" << std::endl;
    return 0;
}
EOF
$ export DISTCC_VERBOSE=1
$ pump distcc g++ -c test.cpp -o test.o
$ g++ -o test test.o
```

Using CMake (example):
```cmake
$ tee > /dev/null CMakeList.txt <<EOF
cmake_minimum_required (VERSION 2.6)
project (Test)
add_executable(Test test.cpp)
```
Compile:
```bash
$ cd <project>
$ mkdir redist && cd redist
$ CC="distcc gcc" CXX="distcc g++" cmake ..
$ pump make -j$(nproc) CC=distcc CXX=distcc
```


