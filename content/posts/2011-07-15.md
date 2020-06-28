---
title: 补丁的制作和使用
date: 2011-07-15T08:00:00+08:00
draft: false
toc:
comments: true
---


## 原理

现在有一个文件file1，通过修改file1得到了文件file2，然后用diff工具比较file1和file2的差异，得到一个补丁文件file.patch，它记录了两个文件的不同之处，patch工具就可以根据这个补丁文件修改file1，从而得到file2。
 
## 相关工具

	diff
	diff [options] 源文件 目标文件

diff用于列出两个文件的不同之处，指示如何由源文件变为目标文件，可以用重定向生成补丁文件，注意：diff只能用于比较文本文件。常用选项：

-c，输出一个基于上下文的diff，即提供每处修改的前后机会内容，这样patch命令可以在打补丁前验证上下文是否匹配，而补丁文件也更容易阅读。

-b，忽略空格引起的变化

-B，忽略插入/删除空行引起的变化

-i，忽略大小写

-N，在比较目录时，如果一个文件只在其中一个目录中找到，那它被视为在第二个目录中是一个空文件

-r，在比较目录时，递归比较所有子目录

-u，使用统一的输出格式
 
	patch
	patch [options]  源文件 补丁文件

patch用于根据补丁文件修改源文件，它会直接改动源文件，打补丁前请注意备份。常用选项：

-R，反向补丁，将已经打了补丁的文件恢复到原来的样子

-p\[num\]，忽略前几层目录，目录的层数由num指定
 
## 例1：比较两个文件

file1:

	this is line1  
	this is line2   
	this is line3  
	this is line4  

file2：

	this is line1  
	this is line2 hello  
	this is line3  
	this is line4  
	this is line5  

执行：

	diff file1 file2 > file.patch  

生成补丁文件file.patch：

	2c2  
	< this is line2   
	---  
	> this is line2 hello  
	4a5  
	> this is line5  

对file1打补丁：

	patch file1 file.patch  

file1的内容就变成了file2，如果想把file1变为原来的样子，执行：

	patch -R file1 file.patch  
 
## 例2：比较两个目录

在工作路径下有两个目录：doc1和doc2。

执行diff命令，生成补丁文件patch：

	diff -Nur doc1 doc2 > doc.patch  

用patch工具为doc1打补丁：

	cd doc1  
	patch -p1 < ../doc.patch  
 
## 参考：

用Diff和Patch工具维护源码： <https://www.ibm.com/developerworks/cn/linux/l-diffp/>
