---
title: netcat 基本用法
date: 2017-07-31T08:00:00+08:00
draft: false
toc:
comments: true
---


netcat 是一个任意 TCP 和 UDP 连接和监听的工具，有时别名也叫 nc 。可以用于各种 TCP 或 UDP 相关的任务，包括打开 TCP 连接，发送 UDP 数据包，监听任意 TCP 和 UDP 端口，同时支持 IPv4 和 IPv6 。

## 1. 语法

    netcat  [option]  ip  port

常用参数：

* -e ，建立链接后执行外部程序。
* -l ，使 netcat 处于监听状态。
* -u ，使用 UDP ，缺省状态下是 TCP 协议。
* -v ，输出详细信息。
* -z ，执行端口扫描。对于 TCP 端口（缺省），尝试在不发送数据的情况下执行连接扫描（完整三路信号握手）。对于 UDP (–u)，缺省情况下会发送空 UDP 包。

## 2. 端口扫描

用于扫描远程主机的某个端口是否处于监听状态，假设一台服务器的端口情况：

    $ netstat -nlp
    (Not all processes could be identified, non-owned process info
     will not be shown, you would have to be root to see it all.)
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    tcp        0      0 0.0.0.0:8883            0.0.0.0:*               LISTEN      19908/mosquitto
    tcp        0      0 0.0.0.0:8884            0.0.0.0:*               LISTEN      19908/mosquitto
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
    tcp6       0      0 :::8883                 :::*                    LISTEN      19908/mosquitto
    tcp6       0      0 :::22                   :::*                    LISTEN      -

可见 TCP 的 22 、8883 和 8884 端口处于监听状态，在本地 PC 上测试 22 端口的连通性：

    $ nc -zv 118.89.16.224 22
    Connection to 118.89.16.224 port 22 [tcp/ssh] succeeded!

连接失败的情况，说明端口没有被监听：

    $ nc -zv 118.89.16.224 80
    nc: connectx to 118.89.16.224 port 80 (tcp) failed: Connection refused

如果要扫描 UDP 端口，加上 -u 参数：

    $ nc -uvz 118.89.16.224 1080
    Connection to 118.89.16.224 port 1080 [udp/*] succeeded!

## 3. 双向通讯

在一台主机上开启监听：

    $ netcat -l 8000

在另一台主机上连接

    $ nc 118.89.16.224 8000

之后在一台主机上输入的信息，就会在另一台主机上显示。

使用 UDP 的例子：

    $ netcat -lu 1080
    $ nc -u 118.89.16.224 1080

