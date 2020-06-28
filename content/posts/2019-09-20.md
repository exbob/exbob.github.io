---
title: Ubuntu 下获取和编译内核源码的方法
date: 2019-09-20T08:00:00+08:00
draft: false
toc: true
comments: true
---



以 Ubuntu 16.04 为例，如果只想获得当前内核版本的源码可以用 `apt-get` ，如果想获取当前系统的所有内核源码建议用 `git` 。

## 1. apt-get

通常是用  `apt-get source` 下载：

```
apt-get source linux-image-$(uname -r)
```

源码会直接下载到当前目录，并自动解压，所以建议在 `/usr/src/` 目录下执行：

```
root@ubuntu:/usr/src# ls
linux-4.4.0                  linux_4.4.0-161.189.dsc  linux-headers-4.4.0-142
linux_4.4.0-161.189.diff.gz  linux_4.4.0.orig.tar.gz  linux-headers-4.4.0-142-generic
```

* linux_4.4.0.orig.tar.gz 是标准内核源码
* linux_4.4.0-161.189.diff.gz 是 Ubuntu kernel 的补丁
* linux_4.4.0-161.189.dsc 是 Debian Source Control 文档，用于描述源码版本的相关信息
* linux-4.4.0 是前两个文件解压打补丁后的源码目录

这样自动下载的小版本号通常是最新的，如果想精确匹配当前的内核版本，可以在 <https://launchpad.net/ubuntu/+source/linux/> 搜索所需的版本，也是这样的三个文件，下载后需要手动解压打补丁：

```
tar xfvz <filname>.orig.tar.gz
gunzip <filename>.diff.gz
cd ./<filename>
patch -p1 -i <file name>.diff 
```

此外，用 `apt-cache search linux-source`  和 `apt-get install linux-source` 也可以搜索并下载当前内核版本的源码，并自动解压到 `/usr/src/` 目录下。

``` shell
root@ubuntu:/usr/src$ ls
linux-headers-4.4.0-116  linux-headers-4.4.0-116-generic  linux-source-4.4.0 linux-source-4.4.0.tar.gz
root@ubuntu:/usr/src$ ls linux-source-4.4.0/
debian debian.master linux-source-4.4.0.tar.gz
```

其中 `debian` 和 `debian.master` 目录下存放的是 ubuntu 发行版中针对内核修改的配置、补丁、说明和一些工具，在 `/usr/src/` 目录下执行如下命令解压，所有源码就解压在 `linux-source-4.4.0` 目录下，：

``` shell
/usr/src$ tar xvf linux-source-4.4.0.tar.gz
```

## 2. git

Ubuntu 使用的内核源码都是用 git 维护的，每个版本的源代码都保存在 `kernel.ubuntu.com`上的 git 存储库中。可以下载你需要的版本：

```
git clone git://kernel.ubuntu.com/ubuntu/ubuntu-<release codename>.git
```

例如下载 16.04 LTS Xenial Xerus 的源码：

```
git clone git://kernel.ubuntu.com/ubuntu/ubuntu-xenial.git
```

这个版本库包含了当前内核源码的所有 commit ，可以检出所需的具体版本。

## 3. 编译

编译内核前，需要安装一些依赖，并复制一份系统自带的配置文件：

``` shell
root@ubuntu:~# apt-get install libncurses5-dev bc libssl-dev
root@ubuntu:~# cp -rf /boot/config-4.4.0-116-generic /usr/src/linux-source-4.4.0/.config
```

如果只是想修改个别模块，必须先复制当前安装的模块的符号表

```
root@ubuntu:/usr/src/linux-4.4# cp -rf /lib/modules/4.4.0-142-generic/build/Module.symvers ./Module.symvers
```

然后执行 `make M=<module dir>` 命令编译。

## 参考

* [Build Your Own Kernel](<https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel>)
* [Kernel Git Guide](<https://wiki.ubuntu.com/Kernel/Dev/KernelGitGuide>)
