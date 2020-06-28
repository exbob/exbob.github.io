---
title: 在 Linux 内核中新增模块代码
date: 2012-12-18T08:00:00+08:00
draft: false
toc:
comments: true
---


## 一个最简单的内核模块

编写一个最简单的内核模块 `hellomod`，源码 `hellomod.c` 如下：

	#include <linux/init.h>
	#include <linux/module.h>
	
	static int __init hellomod_init(void)
	{
		printk("hellomod init\n");
		return 0;
	}
	
	static void __exit hellomod_exit(void)
	{
		printk("hellomod exit\n");
	}
	
	module_init(hellomod_init);
	module_exit(hellomod_exit);
	
	MODULE_LICENSE("GPL");

`Makefile` 文件如下：

	KERDIR=/lib/modules/$(shell uname -r)/build
	PWD=$(shell pwd)
	
	obj-m:=hellomod.o
	
	default:
		make -C ${KERDIR} M=${PWD} modules
	clean:
		make -C ${KERDIR} M=${PWD} clean

直接运行 `make` 即可将其编译为 hellomod.ko 。下面以该模块为例介绍添加到内核的步骤。

## 添加到内核

将一个模块源码添加到内核需要如下三个步骤：

### 1.将源码文件复制的相应的目录中

我们将 `hellomod.c` 复制到内核源码的 `driver/char/` 目录下。

### 2.修改 Makefile 文件

在 `driver/char/Makefile` 文件中添加新的模块：

	obj-$(CONFIG_HELLOMOD) += hellomod.o

其中 `CONFIG_HELLOMOD` 变量是在配置内核是确定的，通常可以取三个值：y 、n 、m 。其中 `CONFIG_` 是固定的， `HELLOMOD` 是自定义的，但必须与 Kconfig 文件中的设置一致。

### 3.修改 Kconfig 文件

`drivers/char/Kconfig` 文件的基本结构如下：

	menu "Characters devices"

	config ...
	...

	source "drivers/char/ipmi/Kconfig"
	...

	endmenu

`menu` 和 `endmenu` 之间定义了一个 `Characters devices` 菜单。`config` 字段定义了一个模块选项，`source` 导入了一个外部的 `Kconfig` 。

在 `menu` 和 `endmenu` 之间添加：

	config HELLOMOD
		tristate "hellomod"
		default m
		help
			just a module

`HELLOMOD` 与 `Makefile` 中定义的变量是一致的。`tristate` 设置的是显示在配置界面的条目，`default` 设置的是该配置的默认值。`help` 显示的是帮助内容。

如上修改之后，运行 `make menuconfig` ，就可以在 `Device Drivers -> Character devices` 中看到 `<M>hellomod` 。退出保存后可以在 `.config` 文件中看到 `CONFIG_HELLOMOD=m` 。

## 新增一个模块目录

如果要为 `hellomod` 新建一个单独的目录，可以按如下步骤完成：

### 1.新建目录并添加源文件

在 `drivers/char/` 目录下新建 `hellomod` 目录，并将 `hellomod.c` 复制到该目录下。

### 2.添加 Makefile

在 `drivers/char/hellomod/` 目录下新建 `Makefile` 文件，内容如下：

	obj-$(CONFIG_HELLOMOD) += hellomod.o

### 3.添加 Kconfig

在 `drivers/char/hellomod/` 目录下新建 `Kconfig` 文件，内容如下：

	menu "hello module"
		config HELLOMOD
			tristate "hellomod"
			default m
			help
				just a module
	endmenu

然后在上层目录 `drivers/chars` 的 `Kconfig` 文件中添加：

	source drivers/char/hellomod/Kconfig
