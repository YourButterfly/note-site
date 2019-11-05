# kvm

## kvm支持

### 硬件

Intel处理器是VT-x，AMD处理器是AMD-V

```shell
lscpu | grep Virtualization
# Virtualization:      VT-x
# Virtualization type: full
```

### 内核支持

kvm和(kvm_intel|kvm_amd)

```shell
lsmod | grep kvm
# kvm_intel             204800  0
# kvm                   593920  1 kvm_intel
# irqbypass              16384  1 kvm
```

## virtio

### virtio内核支持

为客户机提供了一种使用主机上设备的快速有效的通信方式。KVM使用Virtio API作为虚拟机管理程序和客户机之间的连接层，为虚拟机提供准虚拟化设备（亦称Virtio设备）。所有Virtio设备都包括两部分：主机设备和客户机驱动程序。

```shell
lsmod | grep virtio
# 如何加载内核模块
# modprobe module_name 或者 insmod filename [args]
# 如何卸载内核模块
# modprobe -r module_name 或者 rmmod module_name

# 支持的设备列表
# 网络设备 (virtio-net)
# 硬盘设备 (virtio-blk)
# 控制器设备 (virtio-scsi)
# 串口设备 (virtio-serial)
# balloon设备 (virtio-balloon)
```

## 嵌套虚拟化

### 启用

临时

```shell
modprobe -r kvm_intel
modprobe kvm_intel nested=1
```

持久

```shell
cat >> /etc/modprobe.d/modprobe.conf <<EOF
options kvm_intel nested=1
EOF
```

查看

```shell
cat /sys/module/kvm_intel/parameters/nested
# or
systool -m kvm_intel -v | grep nested
```
