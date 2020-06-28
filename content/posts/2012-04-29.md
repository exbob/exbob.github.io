---
title: Vim/Cscope 教程
date: 2012-04-29T08:00:00+08:00
draft: false
toc:
comments: true
---


原文：

The Vim/Cscope tutorial

<http://cscope.sourceforge.net/cscope_vim_tutorial.html>

Translated by Bob

2012-4-27

Email：<gexbob@gmail.com>

Blog：<http://shaocheng.li> 

***

Cscope 是一个非常方便的工具, 它会为你的编辑器（即 Vim ）带来很好的舒适性. 幸运的是, Cscope 已经被 Vim 在内部集成.

这个教程向你介绍了 Vim 内建的 Cscope 支持，以及一套让搜索更方便的方法.

假设你知道使用 vi 风格编辑器的基本知识, 但是不需要任何关于Vim的特定知识 (会简单介绍一些用得到的 Vim 特有功能——如多窗口). 你也无需知道任何关于Cscope的知识: 我们会逐步介绍一些基本的东西.

简而言之，Vim 的 Cscope 支持很像你用过的 Vim 的 ctags 功能。但是因为 Cscope 的搜索类型比 ctags 更多，所以会有些不同。

<!-- more -->

这是一个实践教程, 所以要启动一个 shell , 然后跟着下面的步骤做:

1.

如果你的电脑还没有Cscope，先下载并安装。理想状态下, 你的 Vim 应该是 6.x 版本, 但是用 Vim 5 以上的版本都可以获得大部分功能 (无法使用垂直分割, 但是通过修改相关文件可以使用水平分割).

注意: 如果你的 Vim 版本在编译时没有指定 '--enable-cscope' 选项, 你需要重新编译和配置 Vim 。 大部分随 Linux 发行版安装的 Vim 二进制文件都使能了 Cscope 插件。

2.

下载 cscope_maps.vim 文件, 让 Vim 在启动时读取它. 如果你用的是 Vim 6.x, 把它复制到 $HOME/.vim/plugin 目录下即可 (或者其他插件子目录). 如果你用的是 Vim 5.x, 只能把 cscope_maps 文件的内容复制到 $HOME/.vimrc 文件中, 或者黏贴 "source cscope_maps.vim" 一行到 .vimrc 文件.

3.

进入一个有C代码文件的目录, 执行 'cscope -R' ( '-R' 会让 Cscope 遍历所有子目录, 否则只会检测当前目录). 由于没有使用 '-b' 选项 (它告诉 Cscope 只建立数据库后就退出), Cscope 会打开一个基于 curses 的 GUI 界面. 在这里尝试一些搜索 (提示: 使用方向键可以在搜索类型之间移动, 用 'tab' 键来在搜索类型和搜索结果之间做出选择). 在搜索结果的左边敲入数字, Cscope 就会在 Vim 中打开它的位置. (除非你把 EDITOR 环境变量设为了 Vim 之外的东西). 关闭 Vim, 就会返回到离开 Cscope GUI 时的位置. 有趣。

Cscope 接口有个大问题； 每次要执行新的搜索时都要关闭 Vim. 这就是 Vim 插件. 用 CTRL-D 关闭 Cscope.

4.

启动 Vim. 如果你愿意, 可以在启动时带一个C语言的标识符 (入: 'vim -t main'), 然后就会跳到代码中该标识符定义的地方.

5.

把光标移到一个在程序中多次使用的标识符上. 键入 "CTRL-\ s" (先敲CTRL-\, 然后敲 's') , 应该会在 Vim 窗口的底部看到一个菜单，显示了程序中所有用到这个标识符的地方. 选择其中的一个然后点回车, 就会跳到那里. 类似 ctags, 可以点击 "CTRL-t" 调回搜索之前的位置 (也可以多搜索几个，然后用 CTRL-t 依次释放).

助记符: '\'键在']'键的右边, 用于 ctags 搜索.

6.

尝试同样的搜索, 但是这次用 "CTRL-spacebar s" 组合键. 这一次，Vim 窗口会水平分割为两个, Cscope 的搜索结果将在新的窗口上显示.[如果你从来没用过 Vim 的多窗口: 用 'CTRL-W w' (或 CTRL-W 和方向键, 或 CTRL-W 和 h/j/k/l )在窗口之间移动, 用 'CTRL-W c' (或 ':q')关闭一个窗口, 用 'CTRL-W o'只显示当前窗口, 用 'CTRL-W s' (或 'CTRL-W v' 垂直分割)将窗口分割为两个, 用 ':spl[it] filename]' 在分割的新窗口打开文件。

Mnemonic: 在两个分割的窗口之间会有一条空白的间隔.
 
7.

现在用 "CTRL-spacebar CTRL-spacebar s"(只需按住CTRL连按两次空格)进行同样的搜索. 如果你无法快速的按键, 就打开 cscope_maps.vim 脚本，将 Vim 的超时设置改为注释 [事实上，我通常关闭 Vim 的超时]. 这一次 Vim 的窗口会被垂直分割 (注意: 在 Vim 5.x 上是无法实现的).

8.

到目前为止，我们只用到了'cscope_maps.vim'的热键,所有的搜索都与 Vim 中的光标有关. Cscope 搜索有一个老办法 (用 Vim 的内建 Cscope), 用 ":cscope find symbol foo"命令 (或者简单点，":cs f s foo"). 对于可水平分割的版本,可以用":scscope" (或":scs") . 如果要搜索的词就在光标下，用热键更简单, 命令行的方式可以让你搜索任何标识符，所以要根据实际情况选择.

9.

到目前为止，我们只用了一种搜索方法：‘s'，查找标识符的使用情况。试一下 Cscope 的其他用法: 'g' ，查找全局定义的标识符；'c'，查找一个函数的所有调用；'f' 打开光标所指的文件名 (注意: 因为 Cscope 默认从 /usr/include 查找头文件, 可以用这个功能打开大部分标准头文件). 这些都是我最常用的, 但是还有其他的 (在 cscope_maps.vim 文件可以找到所有的用法，也可以看 Cscope 的 man 手册).

10.

虽然 Cscope 最初只支持 C 语言, 实际上它是个非常灵活的工具，可以支持类似 C++ 和 Java 这样的语言. 你可以把它想象成一个通用的'grep'数据库, 它的作用是分析函数调用和变量定义等类似的结构. Cscope 默认只能解析当前目录(用‘-R’选项可以包括子目录)下的 C, lex, 和 yacc 文件 (.c, .h, .l, .y) , 目前还无法修改文件扩展列表 (我们应该改变这个现状). 所以你必须做一个要解析的文件列表, 命名为'cscope.files'(如果用 'cscope -i foofile'命令，就可以随意命名). 有一个简单（而且十分灵活）的方法是通过 Unix 的'find'命令:

    find . -name '*.java' > cscope.files

然后执行 'cscope -b' 重新构建数据库(‘-b’会让 Cscope 只构建数据库而不启动 GUI), 你就可以浏览 Java 文件中的所有标识符. 很多人用 Cscope 来浏览和编辑大量的文档文件, 说明 Cscope 解析器是非常灵活的。

对于大的项目,你可能需要用 -q 选项, 并且使用更复杂的 'find' 命令. 在 tutorial on using Cscope with large projects 可以得到更多信息.

11.

尝试通过设置 $CSCOPE_DB 环境变量来指定一个你创建的 Cscope 数据库, 这样就不用总是在数据库的目录下启动 Vim. 这种用法多用于代码分别放在多个子目录的项目。注意: 这种情况下, 应该用绝对路径来构建数据库: cd to / ,然后：

    find /my/project/dir -name '*.c' -o -name '*.h' > /foo/cscope.files

然后在 cscope.files 的目录下运行 Cscope (或者用 'cscope -i /foo/cscope.files'), 然后设置并到处 $CSCOPE_DB 变量, 让它指向 cscope.out 文件):

    cd /foo
    cscope -b
    CSCOPE_DB=/foo/cscope.out; export CSCOPE_DB   

(上面最后一条命令是针对 Bourne/Korn/Bash 的: 我已经忘了在基于 csh 的 shell 中如何导出变量, 因为我不喜欢它).

现在应该可以在任何目录下运行 'vim -t foo' ，并且会准确的跳到定义 'foo' 的地方. 我通常会为不同的项目写一个小的 shell 脚本 (仅用于定义和导出 CSCOPE_DB) , 以后只需用一个 'source projectA' 命令来选择它们.

>BUG: 在 Cscope V15.4 之前的版本, 有一个愚蠢的Bug，当执行上面的命令时，可能会导致Vim 停止工作，除非你调用你的数据库是不用默认的‘cscope.out'：用’-f foo'命名为‘foo.out'。

12.

就是这样！如果有什么问题的话就用":help cscope" (Vim) 或 "man cscope" (根据你的 shell) , 还可以学习细微之处.

*Cscope support added to Vim by Andy Kahn*

*Tutorial and cscope_maps.vim by Jason Duell*

*Cscope art by Petr Sorfa*
