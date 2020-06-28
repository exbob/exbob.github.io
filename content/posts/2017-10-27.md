---
title: 用 pyvenv 创建虚拟环境
date: 2017-10-27T08:00:00+08:00
draft: false
toc:
comments: true
---


Mac 默认安装的是 Python2.7 ，再安装一个 Python3.6 ，就出现了两个版本共存的问题，不同的项目、不同的程序要用不同的版本，就需要创建虚拟环境，切换版本。Python3 提供了 pyvenv 模块原生支持虚拟环境，

要创建一个虚拟环境，首先决定一个你想要存放的目录，接着运行 pyvenv 后面携带着目录名:

    pyvenv tutorial-env

如果目录不存在的话，这将会创建一个 tutorial-env 目录，并且也在目录里面创建一个包含 Python 解释器，标准库，以及各种配套文件的 Python “副本”。之后你必须激活它：

    source tutorial-env/bin/activate

激活了虚拟环境会改变你的 shell 提示符，显示你正在使用的虚拟环境，并且修改了环境变量以致运行 python 将会让你得到了特定的 Python 版本，以后用 pip 安装的包都会放在虚拟环境的目录下。例如:

    ~$ python --version
    Python 2.7.10
    ~$ source tutorial-env/bin/activate
    ~(tutorial-env) $ python --version
    Python 3.6.2
    ~(tutorial-env) $ pip --version
    pip 9.0.1 from /Users/lishaocheng/Workspace/py3/lib/python3.6/site-packages (python 3.6)

如果要退回原来的环境，执行 exit 或者再开一个终端即可。

参考：[虚拟环境和包](http://www.pythondoc.com/pythontutorial3/venv.html)
