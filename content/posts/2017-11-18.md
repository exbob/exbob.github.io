---
title: 用 seafile 自建云存储
date: 2017-11-18T08:00:00+08:00
draft: false
toc:
comments: true
---



Seafile 是一款开源的企业云盘，注重可靠性和性能。支持 Windows, Mac, Linux, iOS, Android 平台。支持文件同步或者直接挂载到本地访问，国人开发，官方网站：<https://www.seafile.com> 。

## 1. 部署 Seafile 服务器

我的云服务器是 Ubuntu Server 16.04.1 LTS 64位 ，新建一个工作目录 ~/seafile ，下载最新的服务器安装包 seafile-server_6.2.3_x86-64.tar.gz 到该目录，然后在该目录下做如下工作：

    mkdir installed
    mv seafile-server_6.2.3_x86-64.tar.gz  installed
    tar xvf installed/seafile-server_6.2.3_x86-64.tar.gz

现在的目录结构如下：

    $ tree -L 2
    .
    |-- installed
    |   `-- seafile-server_6.2.3_x86-64.tar.gz
    |-- seafile-server-6.2.3
    |   |-- check_init_admin.py
    |   |-- reset-admin.sh
    |   |-- runtime
    |   |-- seaf-fsck.sh
    |   |-- seaf-fuse.sh
    |   |-- seaf-gc.sh
    |   |-- seafile
    |   |-- seafile.sh
    |   |-- seahub
    |   |-- seahub.sh
    |   |-- setup-seafile-mysql.py
    |   |-- setup-seafile-mysql.sh
    |   |-- setup-seafile.sh
    |   `-- upgrade
    
安装前先确认已经安装了如下包，也可以安装是根据提示补装：

    python 2.7
    python-setuptools
    python-imaging
    python-ldap
    python-urllib3
    sqlite3

然后开始安装：

    cd seafile-server-6.2.3/
    ./setup-seafile.sh

根据提示配置如下选项：

* seafile server name ，seafile 服务器的名称
* seafile server ip or domain ，seafile 服务器的 IP 或者域名
* seafile data dir ，seafile 数据存放的目录
* seafile fileserver port ，seafile 服务使用的 TCP 端口，默认是 8082

安装成功后的目录结构是这样的：

    $ tree -L 1
    .
    |-- ccnet
    |-- conf
    |-- installed
    |-- seafile-server-6.2.3
    |-- seafile-server-latest -> seafile-server-6.2.3
    |-- seahub-data
    `-- seahub.db
    
    6 directories, 1 file

## 2. 启动 Seafile 服务器

在 seafile-server-latest 目录下执行如下命令：

* 启动 seafile 服务： `./seafile.sh start`
* 启动 seahub 服务： `./seahub.sh start <port>` ，默认运行在 8000 端口，第一次启动时会要求新建一个管理员账户

服务启动后, 打开浏览器并输入以下地址:

    http://<server ip>:8000

在打开的登录界面上输入之前创建的管理员帐号的用户名/密码即可。如果不想用 8000 端口，先修改 conf/ccnet.conf 文件中的 SERVICE_URL ，将端口号改为你想要的，比如 8001 :

    SERVICE_URL = http://<server ip>:8001

然后用新的端口号重启服务。

## 3. 开机自启动

创建 seafile 的 systemd 服务文件：

    sudo vim /etc/systemd/system/seafile.service

内容如下，将 ${seafile_dir} 替换为 seafile 安装路径，并且将 user 和 group 指向真正运行 seafile 的用户：

    [Unit]
    Description=Seafile
    # add mysql.service or postgresql.service depending on your database to the line below
    After=network.target
    
    [Service]
    Type=oneshot
    ExecStart=${seafile_dir}/seafile-server-latest/seafile.sh start
    ExecStop=${seafile_dir}/seafile-server-latest/seafile.sh stop
    RemainAfterExit=yes
    User=seafile
    Group=seafile
    
    [Install]
    WantedBy=multi-user.target

创建 seahub 的 systemd 服务文件：

    sudo vim /etc/systemd/system/seahub.service

内容如下，将 ${seafile_dir} 替换为 seafile 安装路径，<port> 替换需要的端口，并且将 user 和 group 指向真正运行 seafile 的用户，：


    [Unit]
    Description=Seafile hub
    After=network.target seafile.service
    
    [Service]
    # change start to start-fastcgi if you want to run fastcgi
    ExecStart=${seafile_dir}/seafile-server-latest/seahub.sh start <port>
    ExecStop=${seafile_dir}/seafile-server-latest/seahub.sh stop
    User=seafile
    Group=seafile
    Type=oneshot
    RemainAfterExit=yes
    
    [Install]
    WantedBy=multi-user.target

使能开机自启动：

    sudo systemctl enable seafile.service
    sudo systemctl enable seahub.service

