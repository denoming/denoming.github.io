---
title: "Install QEMU"
date: 2022-10-15
categories: [Engineering,Linux]
tags: [qemu]
---

# Install

## Installing on Ubuntu Linux

Install packages:
```shell
$ sudo apt install \
qemu-system-common qemu-kvm \
libvirt-daemon-system libvirt-clients bridge-utils \
cpu-checker
$ rehash
```

Check KVM hardware supporting:
```shell
# Check hardware virtualization
$ egrep -c '(vmx|svm)' /proc/cpuinfo
16
# Check supporting KVM acceleration
$ sudo kvm-ok
...
KVM acceleration can be used
```

Add current user to the proper groups:
```shell
$ sudo adduser $USER libvirt
$ sudo adduser $USER kvm
# Reboot
```
After adding current user to groups rebooting is required.

## Installing on Arch Linux

Check KVM hardware supporting:
```shell
$ LC_ALL=C lscpu | grep Virtualization
```
If nothing is displayed after running above command, then your processor does **not** support hardware virtualization.

```shell
$ lsmod | grep kvm
```
If the command returns nothing, the module needs to be loaded manually.

```shell
$ sudo pamac install qemu-full libvirt
```

Add current user to the proper groups:
```shell
$ sudo groupmems -g libvirt -a $USER
$ sudo groupmems -g kvm -a $USER
# Reboot
```
After adding current user to groups rebooting is required.

# Use

Example of QEMU command line:
```shell
$ export KERNEL=...
$ export IMAGE_FILE=...
qemu-system-x86_64 \
-enable-kvm \
-kernel $KERNEL \
-cpu IvyBridge \
-smp 2 \
-m 2G \
-vga std \
-object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-pci,rng=rng0 \
-drive file=$IMAGE_FILE,media=disk,format=raw \
-usb \
-append "root=/dev/sda rw console=ttyS0 nokaslr" \
-serial mon:stdio
```

Specify following environment variables:
* KERNEL - the path to kernel (e.g. bzImage file)
* IMAGE_FILE - the path to root filesystem file (e.g. image-qemux86-64.ext4)

# Configure

## Configure audio (optional)

Check pulse audio socket file availability:
```shell
$ ls -l $XDG_RUNTIME_DIR/pulse/native
srw-rw-rw- 1 denys denys 0 Nov  8 18:42 /run/user/1000/pulse/native
```

Specify QEMU command line to configure backend and frontend devices on virtual machine:
```
-audiodev pa,id=snd0,server=$XDG_RUNTIME_DIR/pulse/native
-device intel-hda
-device hda-output,audiodev=snd0
```

## Configure network virtual bridge (optional)

The virtual bridge provides virtual network and gives an opportunity to communicate the 
host and virtual machines within this network. 

Configure network virtual bridge interface using `libvirt`:
```shell
$ sudo systemctl enable libvirtd
$ sudo systemctl start libvirtd
$ sudo virsh net-autostart --network default
$ sudo virsh net-start --network default
$ ip addr show virbr0
26: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:bd:f7:75 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
$ sudo mkdir -p /etc/qemu
$ sudo tee /etc/qemu/bridge.conf > /dev/null <<EOF
allow virbr0
EOF
```

Make sure QEMU helper script has correct rights:
```shell
$ sudo chown root:root /etc/qemu/bridge.conf
$ sudo chmod 0644 /etc/qemu/bridge.conf
$ sudo chmod u+s /usr/lib/qemu/qemu-bridge-helper
```

Add following line to the QEMU command line in order to add frontend and backend network devices and connect to each other:
```text
-netdev bridge,id=net0,br=virbr0 -device virtio-net-pci,netdev=net0,id=nic1
```
After running QEMU figure out the given IP address using `ifconfig` command. 

To deactivate default bridge:
```shell
$ sudo virsh net-destroy --network default
```

## Configure network bridge (optional)

### Ubuntu Linux

The network bridge enslaves physical interface on your host PC and connects virtual machines to the same network as you interface was connected before.

Assumes your OS is Ubuntu Desktop or any other similar desitrubutions. By default netplan use `NetworkManager` rendered.

#### Configure netplan renderer

Activate mandatory services:
```shell
# On Ubuntu Desktop
$ systemctl enable systemd-resolved
$ systemctl enable NetworkManager
$ systemctl start systemd-resolved
$ systemctl start NetworkManager
$ systemctl is-active systemd-resolved
$ systemctl is-active NetworkManager
# On Ubuntu Not Desktop
$ systemctl enable systemd-resolved
$ systemctl enable systemd-networkd
$ systemctl start systemd-resolved
$ systemctl start systemd-networkd
$ systemctl is-active systemd-resolved
$ systemctl is-active systemd-networkd
```

Configure netplan:
```shell
$ cd /etc/netplan
$ sudo cp 1-network-manager-all.yaml 1-network-manager-all.yaml.backup
$ sudo vim 1-network-manager-all.yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp13s0:
      dhcp4: yes
$ sudo netplan generate
$ sudo netplan apply
$ sudo networkctl 
IDX LINK            TYPE     OPERATIONAL SETUP     
  1 lo              loopback carrier     unmanaged
  2 enp13s0         ether    routable    configured
  3 enx186571fdb57e ether    no-carrier  unmanaged
  4 wlp12s0         wlan     off         unmanaged
  6 docker0         bridge   no-carrier  unmanaged
```
In **not** Ubuntu Dekstop use `networkd` renderer because `NetworkManager` in not available.

We assume the PC is getting IP address from local DHCP server. After applying netplan configuration file we can observe `enp13s0` become `eoutable` and `configured`. Therefore the access to network should become available.

#### Create a bridge with netplan

```shell
$ sudo networkctl 
IDX LINK            TYPE     OPERATIONAL SETUP     
  1 lo              loopback carrier     unmanaged
  2 enp13s0         ether    routable    configured
  3 enx186571fdb57e ether    no-carrier  unmanaged
  4 wlp12s0         wlan     off         unmanaged
  6 docker0         bridge   no-carrier  unmanaged
$ cd /etc/netplan
$ sudo cp 1-network-manager-all.yaml 1-network-manager-all.yaml.backup
$ sudo vim 1-network-manager-all.yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp13s0:
      dhcp4: no
      dhcp6: no
  bridges:
    br0:
      interfaces: [enp13s0]
      dhcp4: yes
      dhcp6: no
$ sudo netplan generate
$ sudo netplan --debug apply
$ sudo networkctl 
IDX LINK            TYPE     OPERATIONAL SETUP
  1 lo              loopback carrier     unmanaged
  2 enp13s0         ether    enslaved    configured
  3 enx186571fdb57e ether    no-carrier  unmanaged
  4 wlp12s0         wlan     off         unmanaged
  6 docker0         bridge   no-carrier  unmanaged
  7 br0             bridge   routable    configured
$ sudo brctl show
bridge name	bridge id		      STP enabled	interfaces
br0		      8000.9af30724daab	no			    enp13s0
docker0	    8000.024239b9144b	no
```
In **not** Ubuntu Dekstop use `networkd` renderer because `NetworkManager` in not available.


Disable netfilter on bridge:
```shell
$ sudo tee /etc/sysctl.d/10-bridge.conf <<EOF
net.bridge.bridge-nf-call-ip6tables=0
net.bridge.bridge-nf-call-iptables=0
net.bridge.bridge-nf-call-arptables=0
EOF
$ sudo tee /etc/udev/rules.d/99-bridge.rules <<EOF
ACTION=="add", SUBSYSTEM=="module", KERNEL=="br_netfilter", RUN+="/sbin/sysctl -p /etc/sysctl.d/10-bridge.conf"
EOF
# After reboot
$ sudo sysctl -a | grep bridge-nf-call
net.bridge.bridge-nf-call-arptables = 0
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
```

To apply changes **you need to reboot**. At last, we configured `br0` as a bridge that is getting IP address from DHCP local server by default. This bridge can be used to make your QEMU virtual machine inside your local network.

#### Configure QEMU to use a bridge

```shell
$ sudo mkdir -p /etc/qemu
$ sudo tee /etc/qemu/bridge.conf > /dev/null <<EOF
allow br0
EOF
$ sudo chown root:root /etc/qemu/bridge.conf
$ sudo chmod 0644 /etc/qemu/bridge.conf
$ sudo chmod u+s /usr/lib/qemu/qemu-bridge-helper
```

Specify br0 as a backend for QEMU virtual machine:
```
-netdev bridge,id=net0,br=br0 -device virtio-net-pci,netdev=net0,id=nic1 \
```
After starting QEMU image with network configuration above you virtual machine should become visible in local network as ordinary network device.

### Arch Linux

Add following lines to `/etc/qemu/bridge.conf` file:
```shell
allow br0
allow br1
```
By these lines we allow particular bridges to be used by QEMU.

Make sure QEMU helper script has correct rights:
```shell
$ sudo chown root:root /etc/qemu/bridge.conf
$ sudo chmod 0644 /etc/qemu/bridge.conf
$ sudo chmod u+s /usr/lib/qemu/qemu-bridge-helper
```

Create particular bridge by `nmcli` tool:
```shell
$ sudo nmcli connection add type bridge ifname br0 stp no
$ sudo nmcli connection add type bridge-slave ifname enp13s0 master br0
$ sudo nmcli connection show --active
NAME                UUID                                  TYPE      DEVICE
bridge-br0          b4659b3c-f2ed-455f-a77f-83ca91400d02  bridge    br0
Wired connection 1  8c5813e4-fa7d-3216-b902-d10e558b5c93  ethernet  enp13s0
lo                  2a15726a-cfa2-4704-8767-b3011a225d6d  loopback  lo
virbr0              5e50fc60-5287-4f69-9cae-6e6ba29f6118  bridge    virbr0
$ sudo nmcli connection down "Wired connection 1"
$ sudo nmcli connection up bridge-br0
$ sudo nmcli connection up bridge-slave-enp13s0
```

Use created bridge in QEMU command line:
```
-netdev bridge,id=net0,br=br0,helper="/usr/lib/qemu/qemu-bridge-helper" -device virtio-net-pci,netdev=net0
```

### Configure libvirt to use a bridge (optional)

```shell
$ tee $HOME/.host-bridge.xml > /dev/null <<EOF
<network>
  <name>host-bridge</name>
  <forward mode="bridge"/>
  <bridge name="br0"/>
</network>
EOF
$ sudo virsh net-list
 Name   State   Autostart   Persistent
----------------------------------------

$ sudo systemctl enable libvirtd
$ sudo systemctl start libvirtd
$ sudo virsh net-define $HOME/.host-bridge.xml
$ sudo virsh net-start host-bridge
$ sudo virsh net-autostart host-bridge
$ sudo virsh net-list --all
 Name          State    Autostart   Persistent
------------------------------------------------
 host-bridge   active   yes         yes
```

# Links

* [QEMU command line invocation documentation](https://www.qemu.org/docs/master/system/invocation.html)