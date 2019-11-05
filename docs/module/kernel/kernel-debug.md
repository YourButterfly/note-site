# debug kernel

## debug ubuntu kernel

### Download kernel debug symbol

调试符号包含源代码级信息，如函数名、函数调用约定和从源代码行号到指令的映射。在调试或分析内核时，这些信息非常有用。

两种方法获取调试符号，一种是编译时带调试符号信息，另一种是下载源代码并在调试时附加，这里使用第二种

```shell
# GPG key import, For Ubuntu 16.04 and higher
# For older distributions:sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys ECDCAD72428D7C01
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C8CAB6595FDFF622
# Add repository config
sudo tee /etc/apt/sources.list.d/ddebs.list << EOF
deb http://ddebs.ubuntu.com/ ${codename}      main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-security main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-updates  main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-proposed main restricted universe multiverse
EOF
# update packages
sudo apt-get update
# Download and install the debugging synbols
codename=$(lsb_release -c | awk  '{print $2}')
sudo apt-get install -y linux-image-$(uname -r)-dbgsym
# Verify
file /usr/lib/debug/boot/vmlinux-$(uname -r)
# debug 
# symbol-file /usr/lib/debug/boot/vmlinux-$(uname -r) in gdb
```

### Download kernel source code

```shell
# Firstly, enable deb-src in /etc/apt/source.list
sudo apt-get update
apt source linux
# 下载的源码不完全与当前系统版本对应
```

尝试

```shell
git clone git://git.launchpad.net/~ubuntu-kernel/ubuntu/+source/linux/+git/$(lsb_release -cs)
# 很慢
```
