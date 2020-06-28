---
title: BitBake 使用笔记
date: 2016-09-12T08:00:00+08:00
draft: false
toc:
comments: true
---



## 1. 简介

BitBake 是用 Python 写的一个程序，它是 OpenEmbedded 构建系统时使用的生产工具，现在有很多嵌入式系统都是在使用，比如Yocto 、WindRiver Linux 等。它是一个多任务引擎，可以并行执行 shell 和 Python 任务，每个任务单元根据预定义的元数据来管理源码、配置、编译、打包，并最终将每个任务生成的文件集合成为系统镜像。例如要从源码构建一个 Linux 系统，需要搭建一个生产环境，然后依次生成 Grub、Kernel、各种库文件、各种可执行文件，然后集合到一个文件系统里。如果你玩过 LFS ，就会了解这个过程的复杂性。BitBake 存在的意义就是提供了一个高效的工具，将这个过程标准化、流程化。BitBake 与 GNU Make 的关系就像 GNU Make 之于 GCC ，运作方式也类似 GNU Make ，又有很多不同：

* BitBake 根据预先定义的元数据执行任务，这些元数据定义了执行任务所需的变量，执行任务的过程，以及任务之间的依赖关系，它们存储在 recipe(.bb)、append(.bbappend)、configuration(.conf)、include(.inc) 和 class(.bbclass) 文件中。
* BitBake 包含一个抓取器，用于从不同的位置获取源码，例如本地文件、源码控制器(git)、网站等。
* 每一个任务单元的结构通过 recipe 文件描述，描述的信息有依赖关系、源码位置、版本信息、校验和、说明等等。
* BitBake 包含了一个 C/S 的抽象概念，可以通过命令行或者 XML-RPC 使用，拥有多种用户接口。

几个概念：

* Recipe 。Recipe 文件是最基本的元数据文件，每个任务单元对应一个 Recipe 文件，后缀是 .bb ，这种文件为 BitBake 提供的信息包括软件包的基本信息（作者、版本、License等）、依赖关系、源码的位置和获取方法、补丁、配置和编译方法、如何打包和安装。
* Configuration 。Configuration 文件的后缀是 .conf ，它会在很多地方出现，定义了多种变量，包括硬件架构选项、编译器选项、通用配置选项、用户配置选项。主 Configuration 文件是 bitbake.conf ，以 Yocto 为例，位于 ./poky/meta/conf/bitbake.conf ，其他都在源码树的 conf 目录下。
* Classes 。Class 文件的后缀是 .bbclass ，它的内容是元数据文件之间的共享信息。BitBake 源码树都源自一个叫做 base.bbclass 的文件，在 Yocto 中位于 ./poky/meta/classes/base.bbclass ，它会被所有的 recipe 和 class 文件自动包含。它包含了标准任务的基本定义，例如获取、解压、配置、编译、安装、打包，有些定义只是框架，内容是空的。
* Layers 。Layer 被用来分类不同的任务单元。某些任务单元有共同的特性，可以放在一个 Layer 下，方便模块化组织元数据，也方便日后修改。例如要定制一套支持特定硬件的系统，可以把与低层相关的单元放在一个 layer 中，这叫做 Board Support Package(BSP) Layer 。
* Append 。Append 文件的后缀是 .bbappend ，用于扩展或者覆盖 recipe 文件的信息。BitBake 希望每一个 append 文件都有一个相对应的 recipe 文件，两个文件使用同样的文件名，只是后缀不同，例如 formfactor_0.0.bb 和 formfactor_0.0.bbappend 。命名 append 文件时，可以用百分号（%）来通配 recipe 文件名。例如，一个名为 busybox_1.21.%.bbappend 的 apend 文件可以对应任何名为 busybox_1.21.x.bb 的 recipe 文件进行扩展和覆盖，文件名中的 x 可以为任何字符串，比如 busybox_1.21.1.bb、busybox_1.21.2.bb ... 通常用百分号来通配版本号。

BitBake 命令的语法可以执行 bitbake -h 查看。-b 用于指定 recipe 文件，-c 用于指定要执行的任务，如果没有指定任务，会按照 recipe 文件完整的执行一次从获取源码到编译打包的过程。要编译一个名为 foo_1.0.bb 的包，可以执行：

    $ bitbake -b foo_1.0.bb                    

可以不用 -b ，而只写包的名字，不加下划线后的版本号和后缀，简化为：

    $ bitbake foo
    
如果要执行清除任务：

    $ bitbake foo -c clean
        
## 2. 工作流程

运行 BitBake 的主要目的是生成一个东西，例如安装包、内核、链接库、或者一个完整的 Linux 系统启动镜像（包括 bootloader、kernel、根文件系统）。当然，你也可以通过使用 bitbake 命令的某些参数，只执行生成过程中的某个步骤，例如编译、获取或清除数据、或者只返回编译环境的信息。

简单说一下使用 BitBake 生成系统镜像的的执行过程。

### 2.1 分析基本元数据

基本元数据由多个文件组成，包括 bblayers.conf 文件（定义项目所需的 layers）、每个 layer 的 layer.conf 文件、以及 bitbake.conf 文件。数据内容有如下几类：

* Recipes：特定软件包的详情。
* Class Data：通用构建信息的抽象总结。
* Configuration Data：针对特定机器的设置，相当于粘合剂，把所有软件集合到一起。

基本元数据都具有全局属性，所有它们对所有的 recipes 都有效。

首先，BitBake 会先搜索当前工作目录下的 conf/bblayers.conf 文件。该文件包含一个 BBLAYERS 变量，它会列出所有项目所需的 layer 的目录。在 BBLAYERS 所列出的 layer 目录中，都会有一个 conf/layer.conf 文件，在这个文件中会有一个 LAYERDIR 变量，它记录了该 layer 的完整路径。这些 layer.conf 文件会自动构建一些关键的变量，例如 BBPATH 和 BBFILES 。BBPATH 记录了 conf 和 classes 目录下的 configuration 和 classes 文件的位置，BBFILES 则用于定位 .bb 和 .bbappdend 文件。如果找不到 bblayers.conf 文件，BitBake 会认为用户已经在环境变量中设置了 BBPATH 和 BBFILES 。

其次，BitBake 会在 BBPATH 记录的位置中寻找 conf/bitbake.conf 文件。该配置文件包含了 

## 参考
[BitBake User Manual](http://www.yoctoproject.org/docs/2.1.1/bitbake-user-manual/bitbake-user-manual.html#bitbake-user-manual)
[Yocto 实用笔记](http://www.kancloud.cn/digest/yocto/138623)
