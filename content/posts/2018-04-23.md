---
title: Libpcap 学习笔记
date: 2018-04-23T08:00:00+08:00
draft: false
toc: true
comments: true
---



## 0. 简介

libpcap 是用于捕获 TCP/IP 网络数据包的 C/C++ 库，支持 Linux 系统，Tcpdump 就是在其基础上开发的，通常用于网络嗅探、数据抓取、协议分析等，官网是 <http://www.tcpdump.org> ，主要的功能有：

* 数据包捕获：捕获流经网卡的原始数据包
* 规则过滤：提供自带规则过滤功能，按需要选择过滤规则
* 流量采集与统计：采集网络中的流量信息
* 自定义数据包发送：构造任何格式的原始数据包

包捕获机制是在数据链路层增加一个旁路处理，并不干扰系统自身的网络协议栈的处理，对发送和接收的数据包通过 Linux 内核做过滤和缓冲处理，最后直接传递给上层应用程序。在 Linux 发行版上可以直接安装 libpcap 或者开发库 libpcap-devel ，也可以下载源码后编译安装。

## 1. 编程

调用 libpcap 库函数前要包含的头文件：
    
    #include <pcap/pcap.h>

调用 libpcap 库抓包的流程：

1. 查找网卡，目的是发现可用的网卡，实现的函数为 `pcap_lookupdev()` 。
2. 获得网卡参数，这里是利用 `pcap_lookupnet()` 函数，获得指定网卡的 IP 地址和子网掩码。
3. 打开网卡，利用第一步的返回值，决定使用哪个网卡，调用 `pcap_open_live()` 将其打开。
4. 编译过滤策略，Lipcap 的重要功能就是提供数据包的过滤，实现的函数是 `pcap_compile()` 。
5. 设置过滤器，调用 `pcap_setfilter()` 函数将编译好的过滤策略设置到相应网卡。
6. 开始捕获数据包，有多种实现函数，具有不同的特性。
7. 关闭网卡，释放资源。

如果是用源码包编译安装的话，在 tests 目录下有几个例程可以参考。

### 1.1. 查找网卡
    
    char errbuf[PCAP_ERRBUF_SIZE];
    int pcap_findalldevs(pcap_if_t **alldevsp, char *errbuf);
    void pcap_freealldevs(pcap_if_t *alldevs);

`pcap_freealldevs()` 函数可以搜索当前系统中的网卡，并构造一个网卡设备链表。如果调用成功，函数返回 0 ，指针 alldevsp 会指向列表的第一个元素；否则返回 -1 ，指针为 NULL ，并在 errbuf 中存放错误信息，errbuf 至少应该是 PCAP_ERRBUF_SIZE 个字节长度的。链表中的每个元素都是 pcap_if_t 类型：

    struct pcap_if {
            struct pcap_if *next;
            char *name;             /* 网卡设备名称，可以调用 "pcap_open_live()" 打开*/
            char *description;      /* 网卡设备描述, 可以为 NULL */
            struct pcap_addr *addresses;  
            bpf_u_int32 flags;      /* PCAP_IF_ interface flags */
    };
    typedef struct pcap_if pcap_if_t;

addresses 是 `struct pcap_addr` 类型的指针，包含了 IP 、掩码等网卡信息：

    struct pcap_addr {
            struct pcap_addr *next;
            struct sockaddr *addr;          /* address */
            struct sockaddr *netmask;       /* netmask for that address */
            struct sockaddr *broadaddr;     /* broadcast address for that address */
            struct sockaddr *dstaddr;       /* P2P destination address for that address */
    };
    typedef struct pcap_addr pcap_addr_t;

bpf_u_int32 flags 是设置网卡状态属性的变量，可选值有：

* PCAP_IF_LOOPBACK ，把网卡设为 loopback 接口
* PCAP_IF_UP ，网卡状态设为 UP
* PCAP_IF_RUNNING ，网卡状态设为 running 

使用完成后，调用如下函数释放设备链表：

    void pcap_freealldevs(pcap_if_t *alldevs);

下面是一个简单的例子：

    #include <stdio.h>
    #include <pcap/pcap.h>
    
    int main()
    {
        int ret = 0;
        int i = 0;
        char errbuf[PCAP_ERRBUF_SIZE] = "\0";
        pcap_if_t *devsp = NULL;
        pcap_if_t *temp = NULL;
    
        printf("start libpcap\n");
        ret = pcap_findalldevs(&devsp, errbuf);
        if(ret==0  && devsp)
        {
            for(temp=devsp;temp;temp=temp->next)
            {
                printf("%d : interface name %s\n",i++,temp->name);
            }
        }
        else
        {
            printf("pcap_findalldevs error : %s\n", errbuf);
        }
        return 0;
    }

编译：

    $ gcc -Wall test.c -o test -lpcap

执行：

    $ ./test
    start libpcap
    0 : interface name eth0
    1 : interface name any
    2 : interface name lo
    3 : interface name nflog
    4 : interface name nfqueue
    5 : interface name usbmon1

### 1.2. 打开网卡

获得了网卡名称后，就可以调用相关函数把它打开：

    pcap_t *pcap_open_live(const char *device, int snaplen, int promisc, int to_ms, char *errbuf);

该函数的作用获得指定网卡的操作句柄，并设置句柄的各种属性，后面的抓包函数都是直接操作这个句柄。各参数的含义：

* device 是网卡名称的字符串
* snaplen 表示对每个数据包，从开头抓取多少个字节，任何一个协议的一个数据包长度都小于 65536 字节，我们可以用这个参数设置只抓取头部，而不关系具体内容
* promise 设为 0 表示关闭混杂模式，1 表示打开，此时网卡也必须打开混杂模式，例如 ifconfig eth0 promisc
* to_ms 用于设置某些抓包函数的超时时间，单位是毫秒，0 表示一直等待
* errbuf 用于存放错误信息

该函数调用成功时会返回一个 pcap_t 类型的指针，失败则返回 NULL 。使用完毕后，应该调用下面的函数将其关闭：

    void pcap_close(pcap_t * p);

如果想要知道某个网卡的类型，可以调用 pcap_datalink() ，它会返回网卡数据链路类型：

    int pcap_datalink(pcap_t *p);

它的返回值列表可以在 <http://www.tcpdump.org/linktypes.html> 查到，常见的有：

LINKTYPE_ name | value | 描述
--- | --- | ---
LINKTYPE_ETHERNET | 1 | IEEE 802.3 以太网
LINKTYPE_PPP | 9 | PPP, 描述在 RFC 1661 和 RFC 1662
LINKTYPE_IEEE802_11 | 105 | IEEE 802.11 无线

### 1.3. 获取数据包

 libpcap 有多种获取数据包的方法，可以根据不同的应用场景进行选择。

    const u_char *pcap_next(pcap_t *p, struct pcap_pkthdr *h);

`pcap_next()` 在获取一个数据包后立即返回，并将数据包存放在返回值指向的内存中，这段内存可能随时被更改或者释放，所以，如果要对数据做进一步处理，最好先拷贝一份，调用失败会返回 NULL 。这种方式处理数据包的内容并不方便，所有我们通常只用这个方法统计数据量，参数 `struct pcap_pkthdr *h` 记录了数据包的其他信息：

    struct pcap_pkthdr
    {
      struct timeval ts;    /* 时间戳 */
      bpf_u_int32 caplen;   /* 本次捕获的数据长度 */
      bpf_u_int32 len;      /* 这个数据包的真实长度 */
    };

因为不能保证每次捕获的包是完整的，例如一个包长 1480，但是捕获到1000的时候，可能因为某些原因就中止了，所以caplen 是记录实际捕获的长度，也就是 1000，而 len 就是1480。

另一个重要的方法是 pcap_loop() ：

    int pcap_loop(pcap_t * p, int cnt, pcap_handler callback, u_char * user);

该函数会连续捕获 cnt 个数据包后退出，如果 cnt < 0 ，会一直循环抓包，直到出现错误或者调用 `pcap_breakloop()` 。在低版本中，没有定义 cnt==0 时的情况，不同平台上的特性无法确定。函数正常退出时返回 0 ，出错返回 -1 ，如果是被 `pcap_breakloop()` 中断的会返回 -2 。每当收到足够的数据包时，`pcap_loop()` 会调用 callback 回调函数，并可以通过 `u_char * user` 向回调函数传递参数，我们可以在回调函数里处理收到的数据包：

    typedef void (*pcap_handler)(u_char *user, const struct pcap_pkthdr *h, const u_char *bytes);  

数据包就存放在指针参数 bytes 指向的内存里，长度可以通过第二个参数 `struct pcap_pkthdr *h` 获取。下面是一个简单的例子：

    #include <stdio.h>
    #include <unistd.h>
    #include <time.h>
    #include <pcap.h>
    
    void get_packet(u_char * arg, const struct pcap_pkthdr * pkthdr, const u_char * packet)
    {
        int * id = (int *)arg;
    
        printf("id: %d\n", ++(*id));
        printf("Packet length: %d\n", pkthdr->len);
        printf("Number of bytes: %d\n", pkthdr->caplen);
        printf("Recieved time: %s\n", ctime((const time_t *)&pkthdr->ts.tv_sec));
        //print packet
        int i;
        for(i=0; i<pkthdr->len; ++i)  {
            printf(" %02x", packet[i]);
            if( (i + 1) % 16 == 0 )
                printf("\n");
        }
        printf("\n\n");
    }
    
    int main()
    {
        char dev[10]="eth0";
        char errbuf[1024];
        int id = 0;
    
        pcap_t* device=pcap_open_live(dev,65535,1,0,errbuf);
        if(!device){
            printf("couldn't open the net device: %s\n",errbuf);
            return 1;
        }
    
        //capture the packet
        pcap_loop(device,-1,get_packet,(u_char*)&id);
    
        return 0;
    }

另一个方法 `pcap_dispatch()` 与 `pcap_loop()` 非常相似，参数完全相同：

    int pcap_dispatch(pcap_t *p, int cnt, pcap_handler callback, u_char *user);

不同的是，这个函数有超时设定，超时时间由 `pcap_open_live()` 函数的第四个参数 to_ms 设置，如果超过 to_ms 毫秒还没有收到数据就会返回。正常情况下，这个函数的返回值是收到的数据包个数，没收到就返回 0 ，出错时返回 -1 ，如果是被 `pcap_breakloop()` 中断的会返回 -2 。有一点要注意，该函数返回后，如果还要继续抓包，可以直接调用 `pcap_dispatch()` 继续抓包，除非调用出错，否则不要关闭设备，频繁的打开关闭设备可能出现意想不到的错误。参数 cnt 表示的是最大值，即最多收到 cnt 个数据包就返回，因为一次只读取一个缓冲区内的数据包，可能会处理少于 cnt 的数据包，如果设为 -1 或者 0 ，表示捕获一个缓冲区内的所有数据包。

这几个函数出错时，可以用 `pcap_geterr()` 获取错误信息：

    char *pcap_geterr(pcap_t *p);

返回值是执行错误信息字符串的指针，最好拷贝一份再做处理。

### 1.3. 过滤器

libpcap 的过滤器是利用 BPF 来筛选网络数据包的，它是类 Unix 系统上数据链路层的一种原始接口，我们可以在过滤器中设置规则，获取想要的网络数据包，比如只获取 UDP 数据包，只获取特定地址的数据包等。过滤数据包需要完成三件事：

1. 构造过滤表达式，这是一个包含筛选规则的字符串
2. 把过滤表达式编译到 BPF
3. 应用过滤器

过滤表达式的语法比较简单，一个过滤表达式通常由一个 id （名字或数字）和多个修饰词组成，修饰词分为三种：

* type ，用来说明 id 是什么类型，可选的 net 、host 和 port 分别表示网络、主机和端口，没有声明时默认是 host ，例如 `net 192.168` , `host 192.168.5.252` ， `port 22`  。
* dir ，表示 id 的传输方向，可选 src 、dst 等，默认是 src or dst ，例如 `dst port 22` 表示目的端口是 22 。
* proto ，限定匹配的协议，比如链路层协议 ether 、wlan ，网络层协议 ip ，传输层协议 tcp 、udp 等。

多个表达式可以用 and 、or 、not 连接，表示逻辑与、或、非。 下面是一些常用的表达式，更多细节可以查看 <https://www.tcpdump.org/manpages/pcap-filter.7.html> :

    #只接受源地址是 192.168.1.1 的数据包
    src host 192.168.1.1
    #只接受目的端口是 8000 的数据包
    dst port 8000
    #只接受 UDP 数据包
    udp 
    #不接受 TCP 数据包
    not tcp
    #只接受 ip 的 ttl==5 的数据包（ip 首部第八个字节是 ttl）
    ip[8]==5
    #接受来自 192.168.5.252 主机，且目的端口是 22 或者 23 的数据包
    dst port 192.168.5.252 and (dst port 22 or dst port 23)

构造好过滤表达式后，就可以调用 `pcap_compile()` 函数进行编译：

    int pcap_compile(pcap_t *p, struct bpf_program *fp, const char *str, int optimize, bpf_u_int32 netmask);

fp 是一个传出参数，用于保存编译后的 BPF ，str 就是过滤表达式，optimize 设为 1 表示对过滤表达式进行优化，netmask 设为 0 即可。编译成功返回 0 ，失败返回 -1 。

> 注意， 在 libpcap 1.8.0 以及之后的版本中，`pcap_compile()` 可以用于多线程，旧版本中，该函数是线程不安全的，如果用于多线程，需要做互斥同步。

编译成功后，调用 `pcap_setfilter()` 应用这个过滤器：

    int pcap_setfilter(pcap_t *p, struct bpf_program *fp);
