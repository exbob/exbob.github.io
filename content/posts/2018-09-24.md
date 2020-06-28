---
title: 用 timedatectl 管理系统时间
date: 2018-09-24T08:00:00+08:00
draft: false
toc:
comments: true
---


查看当前系统时间：

``` shell
$ timedatectl status
      Local time: Thu 2018-09-20 09:42:53 CST
  Universal time: Thu 2018-09-20 01:42:53 UTC
        RTC time: Thu 2018-09-20 01:42:53
       Time zone: Asia/Chongqing (CST, +0800)
 Network time on: yes
NTP synchronized: yes
 RTC in local TZ: no
```

`RTC in local TZ: no` 表示 RTC 时钟不是用本地时间，而是使用 UTC 时间，可以改成使用本地时间，但是并不推荐这么做：

``` shell
$ timedatectl set-local-rtc 1
$ timedatectl
      Local time: Thu 2018-09-20 03:57:58 CEST
  Universal time: Thu 2018-09-20 01:57:58 UTC
        RTC time: Thu 2018-09-20 03:57:59
       Time zone: Europe/Paris (CEST, +0200)
 Network time on: yes
NTP synchronized: yes
 RTC in local TZ: yes

Warning: The system is configured to read the RTC time in the local time zone.
         This mode can not be fully supported. It will create various problems
         with time zone changes and daylight saving time adjustments. The RTC
         time is never updated, it relies on external facilities to maintain it.
         If at all possible, use RTC in UTC by calling
         'timedatectl set-local-rtc 0'.
```

列出所有可用的时区：

``` shell
$ timedatectl list-timezones
```

修改时区：

``` shell
$ timedatectl set-timezone "Europe/Paris"
$ timedatectl
      Local time: Thu 2018-09-20 03:46:02 CEST
  Universal time: Thu 2018-09-20 01:46:02 UTC
        RTC time: Thu 2018-09-20 01:46:02
       Time zone: Europe/Paris (CEST, +0200)
 Network time on: yes
NTP synchronized: yes
 RTC in local TZ: no
$ cat /etc/timezone
Europe/Paris
$ date
Thu Sep 20 03:46:48 CEST 2018
```

设置时间和日期：

``` shell
timedatectl set-time '16:10:40 2018-11-20'
```

关闭时间同步，实际是关闭 `systemd-timesyncd.service` 服务：

``` shell
$ timedatectl set-ntp 0
$ timedatectl
      Local time: Thu 2018-09-20 04:19:41 CEST
  Universal time: Thu 2018-09-20 02:19:41 UTC
        RTC time: Thu 2018-09-20 02:19:41
       Time zone: Europe/Paris (CEST, +0200)
 Network time on: no
NTP synchronized: yes
 RTC in local TZ: no
$ systemctl status systemd-timesyncd.service
● systemd-timesyncd.service - Network Time Synchronization
   Loaded: loaded (/lib/systemd/system/systemd-timesyncd.service; disabled; vendor preset: enabled)
  Drop-In: /lib/systemd/system/systemd-timesyncd.service.d
           └─disable-with-time-daemon.conf
   Active: inactive (dead)
     Docs: man:systemd-timesyncd.service(8)

Feb 11 17:28:02 ubuntu systemd-timesyncd[472]: System clock time unset or jumped backwards, restoring from recorded timestamp:
Sep 19 11:36:39 ubuntu systemd[1]: Started Network Time Synchronization.
Sep 20 03:19:45 ubuntu systemd-timesyncd[472]: Synchronized to time server 91.189.91.157:123 (ntp.ubuntu.com).
Sep 20 04:19:37 ubuntu systemd[1]: Stopping Network Time Synchronization...
Sep 20 04:19:37 ubuntu systemd[1]: Stopped Network Time Synchronization.
Sep 20 04:20:27 ubuntu systemd[1]: Starting Network Time Synchronization...
Sep 20 04:20:27 ubuntu systemd[1]: Started Network Time Synchronization.
Sep 20 04:20:27 ubuntu systemd-timesyncd[1901]: Synchronized to time server 91.189.89.198:123 (ntp.ubuntu.com).
Sep 20 04:20:51 ubuntu systemd[1]: Stopping Network Time Synchronization...
Sep 20 04:20:51 ubuntu systemd[1]: Stopped Network Time Synchronization.
```

`systemd-timesyncd.service` 调用的是 `/lib/systemd/systemd-timesyncd` 守护进程执行时间同步，配置文件是 `/etc/systemd/timesyncd.conf` ，有两个选项：

* NTP= ，主 NTP 服务器域名或者 IP 列表，多个地址之间用空格隔开。
* FallbackNTP= ，备用 NTP 服务器域名或者 IP 列表，多个地址之间用空格隔开。

每次时间同步后都会更新 `/var/lib/systemd/clock` 文件：

``` shell
$ ll /var/lib/systemd/clock
-rw-r--r-- 1 systemd-timesync systemd-timesync 0 Spe 25 14:09 /var/lib/systemd/clock
```

大约 20 分钟同步一次，这个时间间隔不可以设置。
