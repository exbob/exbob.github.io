---
title: Linux系统文本模式下的截屏
date: 2011-07-29T08:00:00+08:00
draft: false
toc:
comments: true
---


Linux系统文本模式下截屏要用到/dev目录下的vcs设备：

	ls  /dev/vcs*  
	/dev/vcs  /dev/vcs1  /dev/vcs2  /dev/vcs3  /dev/vcs4  /dev/vcs5  /dev/vcs6   

其中，/dev/vcs 是当前的虚拟控制台的内容，/dev/vcs1 是tty1的内容。
 
截取当前控制台的内容：

	cat  /dev/vcs  > filename  

得到的文件 filename 是一个文本文件，直接用编辑器即可查看。
 
如果要截取 tty1 的内容，就执行：

	cat  /dev/vcs1  > filename
