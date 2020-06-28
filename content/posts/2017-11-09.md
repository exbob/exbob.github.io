---
title: 用 Supervisor 管理进程
date: 2017-11-09T08:00:00+08:00
draft: false
toc:
comments: true
---


Supervisor 是一个 Python 编写的进程管理工具，可以帮助我们实现进程的启动、关闭和重启，可以对多个进程独立管理，或者分组管理，通常用于 Linux 服务器的进程管理，官方网站 [supervisord.org](http://supervisord.org) 。有两个主要的组成部分：

* supervisord，运行 Supervisor 时会启动一个进程 supervisord，它负责启动所管理的进程，并将所管理的进程作为自己的子进程来启动，而且可以在所管理的进程出现崩溃时自动重启。
* supervisorctl，命令行管理工具，可以用来执行 stop、start、restart 等命令，来对这些子进程进行管理。

## 1. 安装

可以用 pip 安装：
   
    sudo pip install supervisor

如果是 Ubuntu 系统，也可以用 apt-get ：

    sudo apt-get install supervisor

## 2. 配置

安装成功后，需要手动生成一个配置文件，安装包提供了 `echo_supervisord_conf` 工具完成这项工作：

    sudo echo_supervisord_conf > /etc/supervisord.conf

出去注释部分，一些有用的配置选项：

    [unix_http_server]
    file=/tmp/supervisor.sock   ; UNIX socket 文件，supervisorctl 会使用
    
    [supervisord]
    logfile=/tmp/supervisord.log ; 日志文件，默认是 $CWD/supervisord.log
    logfile_maxbytes=50MB        ; 日志文件大小，超出会 rotate，默认 50MB
    logfile_backups=10           ; 日志文件保留备份数量默认 10
    loglevel=info                ; 日志级别，默认 info，其它: debug,warn,trace
    pidfile=/tmp/supervisord.pid ; pid 文件
    nodaemon=false               ; 是否在前台启动，默认是 false，即以 daemon 的方式启动
    minfds=1024                  ; 可以打开的文件描述符的最小值，默认 1024
    minprocs=200                 ; 可以打开的进程数的最小值，默认 200
    
    [rpcinterface:supervisor]
    supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface
    
    [supervisorctl]
    serverurl=unix:///tmp/supervisor.sock ; 通过 UNIX socket 连接 supervisord，路径与 unix_http_server 部分的 file 一致

    ; 包含其他的配置文件
    [include]
    files = relative/directory/*.ini    ; 可以是 *.conf 或 *.ini

在这个文件的最底部，可以添加我们要管理的进程。假设有一个定时打印时间的脚本 test.sh ，在 /home/ubuntu/ 目录下：

    #!/bin/sh
    
    touch test.log
    while true :
    do
        date >> /home/ubuntu/test.log
        sleep 10
    done

我们把它添加到 supervisord ，在配置文件中添加：

    [program:test]
    directory=/home/ubuntu/    ; 程序执行的目录
    command=/home/ubuntu/test.sh    ; 执行程序的命令
    autostart = true     ; 在 supervisord 启动的时候也自动启动
    startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
    autorestart = true   ; 程序异常退出后自动重启
    startretries = 3     ; 启动失败自动重试次数，默认是 3
    user = leon          ; 用哪个用户启动
    redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
    stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
    stdout_logfile_backups = 20     ; stdout 日志文件备份数
    stdout_logfile = /var/log/usercenter_stdout.log      ; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）

其中 [program:test 中的 test 是应用程序的唯一标识，不能重复。对该程序的所有操作（start, restart 等）都通过名字来实现。要注意的是，Supervisor 只能管理在前台运行的程序，所以如果应用程序有后台运行的选项，需要关闭。配置成功后，就可以启动 supervisord ：

    ~$ supervisord -c /etc/supervisord.conf
    ~$ ps -ef | grep super
    ubuntu    3667     1  0 16:52 ?        00:00:00 /usr/bin/python /usr/local/bin/supervisord -c /etc/supervisord.conf
    ~$ ps -ef | grep test
    ubuntu    3668  3667  0 16:52 ?        00:00:00 /bin/sh /home/ubuntu/test.sh

## 3. 管理

Supervisorctl 是 supervisord 的一个命令行客户端工具，通过它实现对程序的启动和停止，可以单独关闭一个程序：

    ~$ supervisorctl stop test
    test: stopped

如果要管理所有程序，用保留的名称 all ：

    ~$ supervisorctl start all
    test: started
    ~$ supervisorctl status test
    test                             RUNNING   pid 4277, uptime 0:00:20

其他的选项：

    reread    ＃ 读取有更新（增加）的配置文件，不会启动新添加的程序
    update    ＃ 重启配置文件修改过的程序

除了 supervisorctl 之外，还可以配置 supervisrod 启动 web 管理界面，这个 web 后台使用 Basic Auth 的方式进行身份认证。除了单个进程的控制，还可以配置 group，进行分组管理。经常查看日志文件，包括 supervisord 的日志和各个 pragram 的日志文件，程序 crash 或抛出异常的信息一半会输出到 stderr，可以查看相应的日志文件来查找问题。更多的信息可以查看官方的帮助文档：<http://supervisord.org/index.html>

## 4. 开机启动

在 /etc/systemd/system 目录下添加 supervisord.service 文件，内容如下：

    cat supervisord.service
    [Unit]
    Description=supervisord
    # add mysql.service or postgresql.service depending on your database to the line below
    After=network.target
    
    [Service]
    Type=oneshot
    ExecStart=/usr/local/bin/supervisord -c /etc/supervisord.conf
    ExecStop=/usr/local/bin/supervisorctl shutdown
    ExecReload=/usr/local/bin/supervisorctl reload
    RemainAfterExit=yes
    User=ubuntu
    Group=ubuntu
    
    [Install]
    WantedBy=multi-user.target

然后执行 `systemctl  enable supervisord.service` 使能该服务开机启动。
