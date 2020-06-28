---
title: 在 ubuntu 中下载内核源码
date: 2013-04-27T08:00:00+08:00
draft: false
toc:
comments: true
---


>下载源码前必须确保已经安装了 `dpkg-dev` 。使用 `apt-get install dpkg-dev` 命令安装。

查找带ubuntu补丁的内核源码

	apt-cache search linux-source

该条命令会查找到很多版本的内核源码，选择所需要的，然后执行：

	apt-get source linux-source-[version]

开始将内核源码包下载到当前目录中。源码包中有三个文件：\*.dsc 、\*.diff.gz 和 \*.orig.tar.gz 。

下载完成后会自动调用 `dpkg-source` 命令，根据 `dsc` 文件中的信息，将源码包解压到同名目录中。然后就可以在源码目录下编译内核了。

>不只是内核源码，用 apt-get 下载其他源码也是这样的方式。

在编译源码包前，还可以用下面命令安装具有依赖关系的相关软件包：

	apt-get build-dep

在源码目录下执行 `dpkg-buildpackage` 命令，会生成 Deb 软件包，并放置在上层目录中。
