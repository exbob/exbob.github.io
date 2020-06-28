---
title: 包管理器 dnf 学习笔记
date: 2018-09-21T08:00:00+08:00
draft: false
toc:
comments: true
---


dnf 是 Fedora 上的新一代软件包管理器，从 Fedora22 开始取代了原有的 yum 。拥有超级用户权限才可以使用 dnf 在系统上安装、更新和删除软件包。

## 1. 配置

dnf 的配置文件位于 `/etc/dnf/dnf.conf` ,文件里的配置信息按不同种类分为多个小节，其中必须的 [main] 小节包含了所有全局选项，还可能包含一些 [repository] 小节用于设置特定存储库的选项，但是建议在 `/etc/yum.repos.d/` 目录中的 `.repo` 文件里定义某个存储库的选项，这些选项的值会覆盖 [main] 小节的同名选项。

最简单的 `/etc/dnf/dnf.conf` 文件类似如下：

    [main]
    gpgcheck=1
    installonly_limit=3
    clean_requirements_on_remove=true

常用的选项和含义包括：

* gpgcheck=value ，value 可选两个值，0 表示禁止所有 GPG 签名检查，1 表示对所有安装的包进行 GPG 签名检查。
* installonlypkgs=space separated list of packages ，这个选项中列出的软件包只安装不更新，多个软件包之间用空格分开。
* installonly_limit=value ，value 是一个整数，表示 installonlypkgs 指令中列出的软件包可以同时安装的最大版本数。installonlypkgs 指令的默认值包括几个不同的内核包，因此，更改 installonly_limit 的值也会影响任何单个内核包的最大安装版本数。 /etc/dnf/dnf.conf 中列出的默认值为 installonly_limit = 3，建议不要降低此值，尤其是低于2。
* keepcache=value ，value 可选两个值，0 表示成功安装后不保留标头和软件包的缓存，这是默认值。1 表示成功安装后保留缓存。



## 2. 使用

### 2.1. 更新

检测并安装所有软件包的更新：

    dnf upgrade

如果带有包的名字就可以单独更新某个包：

    dnf upgrade package_name

只检查可用的更新：

    dnf check-update

### 2.2. 查找

在源里搜索所有名字里包含 `term` 的包：

    dnf search term

名字匹配时支持两种通配符，如果名字本身就包含这两个通配符，可以用反斜杠转义：

* \* ，星号可以匹配任何字符多次
* ？，问号可以匹配任何一个字符

列出所有已安装的和可安装的软件包：

    dnf list all

仅列出已安装的软件包

    dnf list installed

列出未安装但可用的软件包：

    dnf list available

也可以列出名字匹配的包：

    dnf list term

显示某个包的详细信息：

    dnf info package_name

### 2.3. 安装

用 install 命令安装软件包，可以一次安装多个：

    dnf install package_name...

如果不知道某个软件包的名字，只知道它的某个二进制文件的安装路径，也可以只写该文件的路径，dnf 会自动搜索并安装，例如：

    dnf install /usr/sbin/named

删除某个包用 remove 命令：

    dnf remove package_name...

### 2.4. 历史

dnf 的 history 命令可以让用户查看使用记录，包括安装、更新、删除等各种操作使用的命令和时间等详情，例如列出所有的历史操作记录：

    dnf history list

根据操作顺序的 ID 列出某个范围的历史记录：

    dnf history list start_id..end_id

撤销某一步操作：

    dnf history undo id

重做某一步操作：

    dnf history redo id

## 参考

[DNF Command Reference](https://dnf.readthedocs.io/en/latest/command_ref.html>
