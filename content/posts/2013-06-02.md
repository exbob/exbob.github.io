---
title: QNX 编程读写 I/O 端口
date: 2013-06-02T08:00:00+08:00
draft: false
toc:
comments: true
---


要读写 I/O 端口，必须获得 I/O 权限，这需要以 root 权限运行线程，并且调用 ThreadCtl() 函数：

	#include <sys/neutrino.h>
	ThreadCtl( _NTO_TCTL_IO, 0 );

然后调用 `mmap_device_io` 函数映射 I/O 端口地址：

	#include <stdint.h>
	#include <sys/mman.h>
	
	uintptr_t mmap_device_io( size_t len, uint64_t io );

io 表示要访问的 I/O 端口地址（基地址），len 表示要访问的 I/O 端口的字节数。如果调用成功会返回映射后的起始地址，可以直接用 in8 和 out8 等函数访问；如果失败则返回 `MAP_DEVICE_FAILED`。

调用读写 I/O 端口的函数要包括头文件：

	#include <hw/inout.h>

主要的函数有：

	uint8_t in8( uintptr_t port );

从指定端口读取 8 位数据，port 表示端口地址，返回值是读到的数据。

	void out8( uintptr_t port, uint8_t val );

向指定端口写一个 8 位数据。
