---
title: 用 iperf 测试网络性能
date: 2017-08-01T08:00:00+08:00
draft: false
toc:
comments: true
---


iperf 是一个客户端/服务器端工作模式的网络性能测试工具，用于测试 TCP 或者 UDP 的吞吐量，记录延迟、丢包率、MTU等信息。

## 1. 语法
缺省状态下，iperf 使用 TCP 传输协议，服务器端的语法：

    iperf  -s  -p  [port]  [ options ]

加 `-D` 参数可以后台运行。

客户端的语法：

    iperf  -c  [server ip]  -p  [port]  [options]

常用的参数有：

* -f [kmKM] ，分别以 Kbits ， Mbits ， KBytes ， MBytes 格式显示报告，默认是 Mbits 。
* -l ，读写缓存区的大小，默认是 8KB 。
* -w ，设置 TCP 的窗口大小。
* -t ，设置测试时间，默认是 10 秒。
* -i ，打印报告的时间间隔，单位是秒。
* -o ，输出打印信息到问题。
* -u ，设置为 UDP 协议，两端都要加。
* -b n[KM] ，设置使用 UDP 协议时的带宽 n bits/sec ，默认是 1Mbit/sec 。

## 2. 测试 TCP 的带宽

原理比较简单，在客户端和服务端建立三次握手连接后，客户端带宽的大小等于发送的总数据除以发送的总时间。对服务端测得的带宽，则是接收的总数据除以所花时间。服务器端：

    $ iperf -s -p 8000
    ------------------------------------------------------------
    Server listening on TCP port 8000
    TCP window size: 85.3 KByte (default)
    ------------------------------------------------------------
    [  4] local 10.104.231.137 port 8000 connected with 14.197.98.229 port 65237
    [ ID] Interval       Transfer     Bandwidth
    [  4]  0.0-10.0 sec  36.8 MBytes  30.7 Mbits/sec

客户端：

    $ iperf -c 118.89.16.224 -p 8000 -i 2 -t 10
    ------------------------------------------------------------
    Client connecting to 118.89.16.224, TCP port 8000
    TCP window size:  128 KByte (default)
    ------------------------------------------------------------
    [  5] local 192.168.1.104 port 65237 connected with 118.89.16.224 port 8000
    [ ID] Interval       Transfer     Bandwidth
    [  5]  0.0- 2.0 sec  9.25 MBytes  38.8 Mbits/sec
    [  5]  2.0- 4.0 sec  5.75 MBytes  24.1 Mbits/sec
    [  5]  4.0- 6.0 sec  6.50 MBytes  27.3 Mbits/sec
    [  5]  6.0- 8.0 sec  8.38 MBytes  35.1 Mbits/sec
    [  5]  8.0-10.0 sec  6.75 MBytes  28.3 Mbits/sec
    [  5]  0.0-10.0 sec  36.8 MBytes  30.8 Mbits/sec

## 3. 测试 UDP 的性能

客户端可以用 `-u` 设置 UDP 数据流的速率。客户端发送数据时，将根据客户端提供的速率计算数据报发送之间的时延。

客户端还可以指定发送数据报的大小。每个发送的数据报包含一个 ID 号，用来唯一标识报文，服务器端根据该 ID 号来确定数据报丢失和乱序。

当把 UDP 报文大小设置可以将整个报文放入 IP 层的包内时，那么 UDP 所测得的报文丢失数据即为 IP 层包的丢失数据，这提供了一个有效的测试包丢失情况的方法。

数据报传输延迟抖动 (Jitter)的测试由服务器端完成，客户发送的报文数据包含有发送时间戳，服务器端根据该时间信息和接收到报文的时间戳来计算传输延迟抖动。传输延迟抖动反映传输过程中是否平滑。由于它是一个相对值，所以并不需要客户端和服务器端时间同步。

服务器端：

    $ iperf -u -s -p 1080
    ------------------------------------------------------------
    Server listening on UDP port 1080
    Receiving 1470 byte datagrams
    UDP buffer size:  208 KByte (default)
    ------------------------------------------------------------
    [  3] local 10.104.231.137 port 1080 connected with 14.197.98.229 port 58756
    [ ID] Interval       Transfer     Bandwidth        Jitter   Lost/Total Datagrams
    [  3]  0.0-20.0 sec  2.50 MBytes  1.05 Mbits/sec   1.030 ms    0/ 1784 (0%)
    [  3]  0.0-20.0 sec  1 datagrams received out-of-order

客户端：

    $ iperf -u -c 118.89.16.224 -p 1080 -i 5 -t 20
    ------------------------------------------------------------
    Client connecting to 118.89.16.224, UDP port 1080
    Sending 1470 byte datagrams
    UDP buffer size: 9.00 KByte (default)
    ------------------------------------------------------------
    [  5] local 192.168.1.104 port 58756 connected with 118.89.16.224 port 1080
    [ ID] Interval       Transfer     Bandwidth
    [  5]  0.0- 5.0 sec   640 KBytes  1.05 Mbits/sec
    [  5]  5.0-10.0 sec   640 KBytes  1.05 Mbits/sec
    [  5] 10.0-15.0 sec   640 KBytes  1.05 Mbits/sec
    [  5] 15.0-20.0 sec   640 KBytes  1.05 Mbits/sec
    [  5]  0.0-20.0 sec  2.50 MBytes  1.05 Mbits/sec
    [  5] Sent 1785 datagrams
    [  5] Server Report:
    [  5]  0.0-20.0 sec  2.50 MBytes  1.05 Mbits/sec   1.030 ms    0/ 1784 (0%)
    [  5]  0.0-20.0 sec  1 datagrams received out-of-order
