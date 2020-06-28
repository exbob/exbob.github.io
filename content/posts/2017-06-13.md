---
title: Paho JavaScript Client 学习笔记
date: 2017-06-13T08:00:00+08:00
draft: false
toc:
comments: true
---



Paho JavaScript Client 是一个用 JavaScript 写的，基于浏览器的 MQTT Client 库，用于使用 WebSockets 协议连接 MQTT Broker 。

* 官网是 <http://www.eclipse.org/paho/clients/js/> 
* GitHub 是 <https://github.com/eclipse/paho.mqtt.javascript>

## 1. 安装

通过 git 克隆到本地：

    git clone https://github.com/eclipse/paho.mqtt.javascript.git
    
默认克隆的是 master 分支，是正式发布的版本，远程还有一个 develop 分支，用于开发测试，包含了一个基于 maven 构建的简单客户端，可以用于单元测试。首先要手动检出：

    git checkout -b develop remotes/origin/develop

然后再构建测试：

    $ mvn
    $ cd src/tests
    $ mvn test -Dtest.server=iot.eclipse.com -Dtest.server.port=80 -Dtest.server.path=/ws
