---
title: iproute2 学习笔记
date: 2020-01-08T08:00:00+08:00
draft: false
toc: true
comments: true
---


iproute2 是一个 Linux 网络管理工具包，提供了 ip 、tc 、ss 等组件，集成了原有的 ifconfg 、arp 、route 、netstat 等多种命令的功能，并新增了很多特性。

## 1. ip 命令

ip 命令的语法是：

```shell
ip [OPTIONS] OBJECT {COMMAND|help}
```

其中：

* OBJECT 表示操作对象，可选的值有 `{ link | addr | addrlabel | route | rule | neigh | ntable | tunnel | maddr | mroute | mrule | monitor | xfrm | token }` ，常用的对象包括：
  * `link`：网络设备
  * `addr`：设备上的协议（IP或IPv6）地址
  * `addrlabel`：协议地址选择的标签配置
  * `route`：路由表条目
  * `rule`：路由策略数据库中的规则
  * `neigh`: ARP 表
* OPTIONS 表示选项，可选的值有 `{ -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] | -h[uman-readable] | -iec | -f[amily] { inet | inet6 | ipx | dnet | link } | -o[neline] | -t[imestamp] | -b[atch] [filename] | -rc[vbuf] [size] }` 常用的选项包括：
  * `-V，-Version`：显示指令版本信息
  * `-s，-stats，statistics`：输出详细信息
  * `-h，-human，-human-readable`：输出人类可读的统计信息和后缀
  * `-o，-oneline`：将每条记录输出到一行，用 `\ `字符替换换行符

### 1.1 配置网卡

调用 `ip addr` 可以查看所有网卡的信息和配置，也可以在后追加网卡的接口名称，单独查看特定网卡，以 `enp1s0` 为例：

```shell
root@localhost:~# ip addr show enp1s0
3: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:1d:f3:52:b7:94 brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.242/24 brd 192.168.5.255 scope global enp1s0
       valid_lft forever preferred_lft forever
    inet6 fe80::21d:f3ff:fe52:b794/64 scope link 
       valid_lft forever preferred_lft forever
```

每个字段的含义：

* `<BROADCAST,MULTICAST,UP,LOWER_UP>` : `BROADCAST`表示该接口支持广播；`MULTICAST`表示该接口支持多播；`UP`表示该网络接口已启用；`LOWER_UP`表示网络电缆已插入，设备已连接至网络。
* `mtu 1500`：最大传输单元（链路层最大一次传输的上层数据大小，也就是一个 IP 包的最大值）为 1500 字节。
* `qdisc mq`：用于数据包排队 。
* `state UP`：网络接口已启用，如果接口没有启用，会显示 DOWN 。
* `qlen 1000`：传输队列长度。
* `link/ether 00:1d:f3:52:b7:94` : 网卡的 MAC 地址。
* `brd ff:ff:ff:ff:ff:ff` : 广播 MAC 地址。
* `inet 192.168.5.242/24` : 网卡的 IPv4 地址。
* `brd 192.168.5.255` : 广播 IPv4 地址。
* `scope global`：全局有效，如果显示为 `link` 表示仅对此设备有效。
* `enp1s0` : 如果前面有 `dynamic` 字符，表示这个网卡的 IP 是动态获取的。
* `valid_lft forever` :  IP 的有效期是永久，如果是动态获取的 IP ，会显示剩余的有效时间。
* `preferred_lft forever` :  首选 IP 的地址的有效期。

为网卡添加或者删除一个 IP 地址：

```shell
ip addr add/del 192.168.0.123/24 dev enp1s0
```

启动或者禁用一个网卡：

```shell
ip link set enp1s0 up/down
```

设置网卡的其他特性也可以用 `ip link set` 语法。例如：

* `arp on/off` : 改变网卡的 NOARP 标识，设为 off 表示禁用该网卡的 ARP 协议。网卡处于 UP 状态时不能执行这个操作。
* `multicast on/off` : 改变网卡的 MULTICAST 标识，设为 off 表示禁用该网卡的组播。
* `dynamic on/off` : 改变网卡的 DYNAMIC 标识，设为 off 表示禁止该网卡动态获取 IP。
* `mtu NUMBER` : 设置该网卡的最大传输单元。

查看网卡的流量统计：

```shell
root@localhost:~# ip -s -s link ls enp1s0
3: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:1d:f3:52:b7:94 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    42861012   137710   0       0       0       151     
    RX errors: length   crc     frame   fifo    missed
               0        0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    520406     6159     0       0       0       0       
    TX errors: aborted  fifo   window heartbeat transns
               0        0       0       0       2       
```

### 1.2 配置路由表

显示系统的路由表：

```shell
root@localhost:~# ip route show
default via 10.76.141.9 dev enp0s29u1u4c2  proto static  metric 100 
10.76.141.8/29 dev enp0s29u1u4c2  proto kernel  scope link  src 10.76.141.12  metric 100 
192.168.5.0/24 dev enp1s0  proto kernel  scope link  src 192.168.5.242  metric 100 
192.168.99.0/24 dev wlp3s0  proto kernel  scope link  src 192.168.99.1  metric 600 
```

* `default` 一行表示缺省路由。

* `via 10.76.141.9` 表示网关，也就是吓一跳的 IP 。

* 普通路由的第一个字段表示目标网段。

* `dev enp1s0` 表示出口网卡。

* `metric 100` 表示优先级，数字小的优先。

添加一条到主机的静态路由：

``` shell
ip route add 192.0.2.1 via 192.168.145.2 dev enp1s0
```

* `add` 表示添加，如果要删除可以用 del 命令
* `192.0.2.1` 表示目标主机的 IP
* `via 192.168.145.2` 表示吓一跳地址
* `dev enp1s0` 表示出口的网卡设备

如果要添加到网络的静态路由，将目标地址设为一个网段即可：

``` shell
ip route add 192.0.2.1/24 via 192.168.145.2 dev enp1s0
```

如果要添加一条默认路由，将目标地址设为 default 即可：

``` shell
ip route add default via 192.168.145.2 dev enp1s0
```

### 1.3 配置 ARP

查看系统的 ARP 缓存：

```shell
root@localhost:~# ip neigh show
192.168.5.252 dev enp1s0 lladdr 8c:a6:df:0c:7a:0a REACHABLE
192.168.5.36 dev enp1s0 lladdr 98:ee:cb:a9:c1:98 STALE
192.168.5.50 dev enp1s0 lladdr f8:7b:20:ee:d2:47 STALE
192.168.5.64 dev enp1s0 lladdr 74:27:ea:6f:08:35 STALE
10.76.141.9 dev enp0s29u1u4c2 lladdr 4c:54:99:45:e5:d5 REACHABLE
fe80::4e54:99ff:fe45:e5d5 dev enp0s29u1u4c2 lladdr 4c:54:99:45:e5:d5 router REACHABLE
```

每一行都是一条 ARP ，以第一行为例：

* `192.168.5.252` 表示设备的 IP 。
* `dev enp1s0` 表示发现此设备的网卡。
* `lladdr 8c:a6:df:0c:7a:0a` 表示此设备的 MAC 地址。
* `REACHABLE` 表示这条 ARP 条目的状态：
  * `REACHABLE` 表示条目有效，设备在线。
  * `STALE` 表示条目有效，但是设备未必在线。
  * `FAILED` 表示条目无效，设备不在线。
  * `PERMANENT` 表示此条目永久有效，直到管理员将其删除，通常是手动添加的静态 ARP 绑定。

添加一条静态 ARP ：

```shell
ip neight add dev enp1s0 add 192.168.5.50 lladdr f8:7b:20:ee:d2:47 nud permanent
```

如果 ARP 表中已经存在了这样的 ARP 映射，执行 add 命令会报错，可以用 replace 将其替换，其他参数不变：

```shell
ip neight relpace dev enp1s0 add 192.168.5.50 lladdr f8:7b:20:ee:d2:47 nud permanent
```

删除一条 ARP 时不用这么多参数，只需指定接口和 IP 即可：

```shell
ip neight delete dev enp1s0 add 192.168.5.50
```

清空当前 ARP 表：

```shell
ip neight flush dev enp1s0
```

## 2. ss 命令

ss 是查看 socket 连接信息的工具。常用的参数有：

* -a ，显示所有 socket 连接的信息。
* -4 ，只显示 IPv4 的 socket 。
* -6 ，只显示 IPv6 的 socket 。
* -t ，只显示 TCP 协议的 socket 。
* -u ，只显示 UCP 协议的 socket 。
* -x ，只显示 Unix 域的 socket 。
* -l ，显示处于 LISTEN 状态的 socket 。
* -o ，显示定时器信息。
* -p ，
* -e ，


## 参考：

* <http://www.policyrouting.org/iproute2.doc.html>

* <https://linux.cn/article-3144-1.html>

  

