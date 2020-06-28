---
title: VScode 使用笔记
date: 2017-11-07T08:00:00+08:00
draft: false
toc:
comments: true
---


## 1. Python 开发环境

为 vscode 安装 [Python](https://marketplace.visualstudio.com/items?itemName=donjayamanne.python) 插件，然后选择 Python 版本：

1. 通过 shift+command+p 组合键打开命令控制板
2. 选择 Python:Select Workspace Interpreter 
3. 选择 Python 版本

## 2. 在终端打开 vscode

在 zsh 终端里用 vscode 直接打开当前目录，编辑配置文件 ~/.zshrc ，在最后加一行：

    alias code='/Applications/Visual\ Studio\ Code.app/Contents/Resources/app/bin/code'

保存后重启终端，找一个项目文件夹，直接打开 vscode ：

    $ code  .

## 3. 远程 FTP 同步

ftp-sync 是一个自动将本地工作目录文件同步到远程 FTP/SFTP 服务器的插件。安装后可以通过三个命令完成同步：

* Ftp-sync: Init，在 .vscode 目录下新建一个默认的 ftp-sync 配置文件
* Ftp-sync: Sync Local to Remote，同步本地文件到远程服务器
* Ftp-sync: Sync Remote to Local，同步远程服务器的文件到本地目录

## 4. Align 插件

这个插件可以对选中的代码进行对齐操作，选中几行代码，按下 command+control+a 组合键即可。

## 5. Markdown

VScode 原生支持 Markdown 预览，快捷键 command+k 再按 v 就可以分屏预览。而 [Markdown preview enhanced](https://shd101wyy.github.io/markdown-preview-enhanced/#/zh-cn/) 插件可以提供更好的预览效果和写作体验，并且可以导出各种格式，包括 HTML、PDF、Word、eBook 等。

## 6. 自定义颜色主题

默认的颜色主题都安装在安装路径下的  `resources/app/extensions/` 目录下，安装的扩展都放在 `C:/Users/[username]/.vscode/extensions/` 目录下，我们可以从默认的颜色主题复制一份到扩展安装路径下，然后在此基础上修改。比如复制一份 theme-solarized-dark ，重命名为 theme-solarized-dark-custom 。

如果是 MacOS ，扩展安装目录在 `~/.vscode/extensions/` 。
