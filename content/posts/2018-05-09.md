---
title: systemd 的网络管理
date: 2018-05-09T08:00:00+08:00
draft: false
toc: true
comments: true
---



## 0. 简介

systemd 是 [freedesktop](https://www.freedesktop.org) 的项目，官网 <https://www.freedesktop.org/wiki/Software/systemd/> ,项目源码在 github 上发布，可以在 <https://github.com/systemd/systemd> 查看所有版本更新、 Bug Fix 和版本对应的文档等。

systemd-networkd 是 systemd 默认提供的网络管理服务，可以完全管理以太网，对于无线网卡，还需要其他服务支持，比如管理 Wi-Fi 的 wpa_supplicant@.service ，管理 PPP 的 ppp@.service 。

管理网卡前，应该确保各网卡的驱动都已经正常加载，systemd 的 systemd-modules-load.service 负责在系统启动时静态加载内核模块。它会从以下路径搜索可用的配置文件：

    /etc/modules-load.d/*.conf
    /run/modules-load.d/*.conf
    /usr/lib/modules-load.d/*.conf

配置文件的内容就是一个内核模块名称的列表，可以用井号 # 或者分号  ;  注释单个模块。

## 1. 基本配置

我的系统中，systemd 的版本是 216 ，有两个以太网卡和一个 Wi-Fi 网卡，先查看一下网卡列表，再查看 systemd-networkd.service 的状态：

    ~# networkctl list
    IDX LINK             TYPE               OPERATIONAL SETUP     
    1 lo               loopback           carrier     unmanaged 
    2 wlp1s0           wlan               off         unmanaged 
    3 enp0s20f6        ether              routable    configured
    4 enp0s20f7        ether              no-carrier  configured
    4 links listed.
    ~# systemctl status systemd-networkd 
    a—? systemd-networkd.service - Network Service
    Loaded: loaded (/lib/systemd/system/systemd-networkd.service; enabled)
    Active: active (running) since Fri 2018-04-27 17:49:22 UTC; 24min ago
        Docs: man:systemd-networkd.service(8)
    Main PID: 260 (systemd-network)
    Status: "Processing requests..."
    CGroup: /system.slice/systemd-networkd.service
            a””a”€260 /lib/systemd/systemd-networkd

    Apr 27 17:49:21 quark systemd[1]: Starting Network Service...
    Apr 27 17:49:22 quark systemd-networkd[260]: lo              : gained carrier
    Apr 27 17:49:22 quark systemd[1]: Started Network Service.
    Apr 27 17:49:22 quark systemd-networkd[260]: eth1            : renamed to enp0s20f7
    Apr 27 17:49:22 quark systemd-networkd[260]: eth0            : renamed to enp0s20f6
    Apr 27 17:49:23 quark systemd-networkd[260]: enp0s20f6       : gained carrier
    Apr 27 17:49:23 quark systemd-networkd[260]: enp0s20f7       : gained carrier
    Apr 27 17:49:25 quark systemd-networkd[260]: enp0s20f6       : lost carrier
    Apr 27 17:49:26 quark systemd-networkd[260]: enp0s20f7       : lost carrier

sytemd-network.service 的配置文件可以位于 /usr/lib/systemd/network/ 或者 /etc/systemd/network/ 目录下，后者具有最高优先级。配置文件有三种类型：

1. .network 文件，设置网卡的 IP 等各项属性
2. .netdev 文件，新建一个虚拟网卡
3. .link 文件，每当一个网卡出现时，udev 都会查找与它同名的 .link 文件

这几类文件都遵循下面的规则：

* 各选项的值都支持星号 * 通配符
* 当 [Match] 段内的条件都匹配时，后面的配置项才会被激活
* 如果 [Match] 段为空，表示后面的配置项在任何情况下都可用
* 无论配置文件在哪个目录，都会统一安装字典顺序进行加载
* 同名文件可以相互替换

如果要给 enp0s20f6 配置一个静态 IP ，可以在 /etc/systemd/network/ 目录下新建一个 eth0.network 文件，内容如下：

    [Match]
    Name=enp0s20f6  # 匹配名为 enp0s20f6 的网卡
    [Network]
    DHCP=none    # 关闭 DHCP 客户端，
    Address=192.168.5.242/24  # 设置 IP 和子网掩码
    Gateway=192.168.5.50   # 设置网关，这项设置会将该网卡添加到缺省路由
    DNS=8.8.8.8  # 设置 DNS 

如果要使用 DHCP 自动获取 IP ，也将 DHCP 设为如下值：

* v4 ，只接受 ipv4 的 IP
* v6 ，只接受 ipv6 的 IP
* both ，同时接受 ipv4 和 ipv6 格式的 IP

启动 DHCP 客户端后，Gateway 和 DNS 也会自动获取，有时我们不希望这样，可以在配置文件中添加一个 [DHCP] 段，做如下设置：

    [DHCP]
    UseDNS=false # 不使用 DHCP 分配的 DNS ，默认值是 true
    UseRoutes=false # 不会将本网卡设为缺省路由，默认值是 true 

在这里会出现一个 Bug ，就是 UseRoutes 设置无效，高版本中已经解决，解决方案在 <https://github.com/systemd/systemd/pull/3075> 。

## 2. Wi-Fi 配置

Wi-Fi 连接还是使用 wpa_supplicant 程序，systemd 提供了 wpa_supplicant@.service 管理整个流程，然后再用 systemd-networkd.service 配置 IP 等。

wpa_supplicant@.service 的内容：

    [Unit]
    Description=WPA supplicant daemon (interface-specific version)
    Requires=sys-subsystem-net-devices-%i.device
    After=sys-subsystem-net-devices-%i.device

    # NetworkManager users will probably want the dbus version instead.

    [Service]
    Type=simple
    ExecStart=/usr/sbin/wpa_supplicant -c/etc/wpa_supplicant/wpa_supplicant-%I.conf -i%I

    [Install]
    Alias=multi-user.target.wants/wpa_supplicant@%i.service

这是 service 模板，启动时在 `@` 符号之后加入 Wi-Fi 网卡的名称使其实例实例化，这个名词还会关联到 wpa_supplicant 的配置文件，我们先在 /etc/ 下新建一个 wpa_supplicant 目录，然后新建一个 wpa_supplicant-wlp1s0.conf 文件，内容如下：

    ctrl_interface=/var/run/wpa_supplicant
    ctrl_interface_group=wheel
    update_config=1
    eapol_version=1
    ap_scan=1
    fast_reauth=1

用 wpa_passphrase 添加无线路由器配置：

    ~# sudo wpa_passphrase [essid] [password] >> /etc/wpa_supplicant/wpa_supplicant-wlp1s0.conf

在 /etc/systemd/network/ 目录下添加配置文件 wireless.network ，内容如下：

    [Match]
    Name=wlp1s0
    [Network]
    DHCP=yes

最后重启各项服务：

    ~# systemctl restart systemd-networkd
    ~# systemctl restart wpa_supplicant@wlp1s0
    ~# systemctl restart systemd-resolved

## 3. 3G/4G 配置

依然使用 pppd 进行 3G/4G 拨号，systemd 提供了 ppp@.service 服务管理 pppd 。首先在 /etc/ppp/peers/ 下建好 ppp 拨号文件 me909s ，然后启动服务：

    ~# systemctl start ppp@me909s 

## 4. 关闭 IPv6

向内核参数加入 ipv6.disable=1 可以完全关闭 IPv6 功能。此外，只加入 ipv6.disable_ipv6=1 内核参数可以保留 IPv6 功能，但不会向网卡分配 IPv6 地址，做法是：

    ~# echo 1 > /proc/sys/net/ipv6/conf/enp0s20f6/disable_ipv6

## 5. ssh server

目前流行的启动 ssh server 的方式是调用 sshd.socket ：

    ~# systemctl start sshd.socket

旧有的 sshd.service 模式会在后台保持一个 sshd 的守护进程，每当有 ssh 连接要建立时，就创建一个新进程，比较适合 SSH 下有大量流量的系统；

新的 sshd.socket 方式也是在每次要建立新的ssh连接时生成一个守护进程的实例，不过监听端口则是交给了 systemd 来完成，意味着没有 ssh 连接的时候，也不会有 sshd 守护进程运行，大部分情况下，使用 sshd.socket 服务更为合适。这也与 MacOS 下的行为相一致，默认只监听端口，有连接时才创建进程。

另外，通过使用 .socket 文件来管理需要监听端口的服务，可以直接通过 systemctl 来查看一些网络相关的信息，如监听的端口、目前已经接受的连接数、目前正连接的连接数等。

## 参考 

* [How to use systemd-networkd to manage your wifi](https://forum.manjaro.org/t/how-to-use-systemd-networkd-to-manage-your-wifi/1557)
* [Systemd-network](https://wiki.archlinux.org/index.php/Systemd-networkd)
