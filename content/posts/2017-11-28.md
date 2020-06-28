---
title: Linux 系统的 USB 设备结构
date: 2017-11-28T08:00:00+08:00
draft: false
toc:
comments: true
---


以一台 x86 主机为例，用 lsusb 可以查看它的所有 USB 设备：

    # lsusb
    Bus 001 Device 003: ID 1bc7:0021 Telit HE910
    Bus 002 Device 002: ID 04e2:1410 Exar Corp. 
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 002 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub

在 sys 文件系统的 /sys/bus/usb/devices/ 目录下可以看到所有 USB 设备的树形结构：

    /sys/bus/usb/devices# ls -l
    total 0
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-0:1.0 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-0:1.0
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.0 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.0
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.1 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.1
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.10 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.10
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.11 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.11
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.12 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.12
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.13 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.13
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.2 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.2
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.3 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.3
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.4 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.4
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.5 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.5
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.6 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.6
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.7 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.7
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.8 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.8
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 1-2:1.9 -> ../../../devices/pci0000:00/0000:00:14.3/usb1/1-2/1-2:1.9
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 2-0:1.0 -> ../../../devices/pci0000:00/0000:00:14.4/usb2/2-0:1.0
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 2-1 -> ../../../devices/pci0000:00/0000:00:14.4/usb2/2-1
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 2-1:1.0 -> ../../../devices/pci0000:00/0000:00:14.4/usb2/2-1/2-1:1.0
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 2-1:1.1 -> ../../../devices/pci0000:00/0000:00:14.4/usb2/2-1/2-1:1.1
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 usb1 -> ../../../devices/pci0000:00/0000:00:14.3/usb1
    lrwxrwxrwx 1 root root 0 Nov 26 10:57 usb2 -> ../../../devices/pci0000:00/0000:00:14.4/usb2

usb1 和 usb2 是 PCI 总线上的两个 USB controller ，也就是两个 root hub ，它们扩展出两条 USB 总线，而其他设备都是这两条总线下的子设备，它们的命名格式是 `根集线器号-集线器端口号:配置.接口`，进入 usb2/1-2 目录，查看设备号：

    /sys/bus/usb/devices/usb1# cd 1-2/
    /sys/bus/usb/devices/usb1/1-2# cat devnum 
    3

可以确认，1-2 就是设备 `Bus 001 Device 003: ID 1bc7:0021 Telit HE910` 的文件夹，这个设备号是 USB 总线枚举设备时自动分配的，如果拔掉设备重插，设备号会重新分配，自动向后增长。设备目录下还有一些有用的文件：

* busnum：设备接入的 USB 总线号
* idProduct：设备 ID
* idVendor：厂商 ID

参考：

* <http://www.linux-usb.org/FAQ.html>
* <https://www.kernel.org/doc/Documentation/ABI/stable/sysfs-bus-usb>
