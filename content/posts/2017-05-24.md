---
title: Linux Test Project 学习笔记
date: 2017-05-24T08:00:00+08:00
draft: false
toc:
comments: true
---


## 1. 简介

Linux Test Projec 是一个开源项目，简称 LTP ，致力于提供一套测试工具，用于测试和验证 Linux 内核的功能和稳定性，官网地址：<https://linux-test-project.github.io>。

## 2. 安装

安装前要准备 git、gcc、automake、autoconf、m4、bison、byacc、flex 。然后从 github 克隆源码：

    $ git clone https://github.com/linux-test-project/ltp.git
    $ cd ltp

配置、编译、安装，文件都安装在 /opt/ltp/ 目录下。：

    $ ../configure
    $ make all
    $ make install
    $ cd /opt/ltp/
    $ ls
    bin  IDcheck.sh  runltp  runltplite.sh  runtest  scenario_groups  share  testcases  testscripts  ver_linux  Version

## 3. 结构说明

整套测试工具分为三大部分：测试脚本、测试驱动和测试用例。测试脚本位于 testscripts/ 目录下，包含了文件系统、磁盘、内核等各种测试项目的脚本；测试驱动位于 bin/ 目录下，主要是一些测试用的辅助脚本，比如 ltp-pan 和 ltp-scanner ；测试用例位于 testcase/ 目录下。其他各目录和文件的功能：

* IDcheck.sh ，检查系统是否缺少执行 LTP 测试套件所需的用户和用户组，如果缺少，就自动创建。
* runltp ，执行 LTP 测试套件的脚本。
* runltplite.sh ，用于测试 LTP 安装，也可用于执行单项测试。
* runtest ，测试驱动，用户连接测试脚本和测试用例。
* share ，测试脚本的使用说明。
* ver_linux ，打印当前系统各项信息的脚本。

LTP 的执行原理是从一个测试脚本中读取所测条目需要执行的命令行，然后等待该项测试的结束，并记录详细的测试输出。简单地说，LTP 测试套件通过执行测试脚本 runltp (或着 runltplite.sh，testscripts 内的测试脚本），调用驱动程序 ltp-pan 执行 testcases 内的测试项目，输出测试结果，并利用 ltp-scanner 整理数据。默认状态下 ltp-pan 会随机的选择一个命令行来运行，可以指定在同一时间要执行测试的次数。ltp-pan 会记录测试产生的详细的格式复杂的输出，但它不进行数据的整理和统计，数据整理统计的工作由 ltp-scanner 来完成，ltp-scanner 是一个测试结果分析工具，它会理解 ltp-pan 的输出格式，并通过表格的形式总结测试 passed 或 failed 的情况。

## 测试套件的使用

LTP 提供了两个 runltp 和 runltplite.sh 。runltp 用于验证内核。这个脚本串行地运行一组测试，并报告全部结果。默认地，这个脚本执行：

* 文件系统压力测试。 
* 硬盘 I/O 测试。 
* 内存管理压力测试。 
* IPC 压力测试。
* SCHED 测试。 
* 命令功能的验证测试。
* 系统调用功能的验证测试。

LTP 提供的测试内容不完全写入 runltp ，测试时可以根据需求修改 runltp 并添加内容。
