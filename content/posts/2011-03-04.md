---
title: 判断进程是否存在的脚本
date: 2011-03-04T08:00:00+08:00
draft: false
toc:
comments: true
---


test.sh :

	#!/bin/bash  
	pid=`ps -ef | grep -v grep | grep -v "test.sh" | grep $1 | sed -n  '1P' | awk '{print 	$2}'`  
	if [ -z $pid ] ; then  
	    echo "no this process"  
	else  
	    echo $pid  
	fi  

例：

查询是否存在hello进程

	#./test.sh   hello

如果存在 hello 进程，会输出 PID ; 不存在会输出 no  this  process
