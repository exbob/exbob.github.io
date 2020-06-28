---
title: 神级排序算法
date: 2011-06-28T08:00:00+08:00
draft: false
toc:
comments: true
---


下面是一个排序算法，用shell实现的：

sleepsort.sh

	#!/bin/bash   
	
	function f()   
	{       
		sleep "$1"      
		echo "$1"  
	}   
	while [ -n "$1" ]   
	do      
	f "$1" &       
	shift  
	done  
	wait   


用法：
 
	./sleepsort.sh 5 3 6 3 6 3 1 4 7

这个算法太NB、太BT、太搞笑了！

神马冒泡、插入、归并...  全是浮云啊！

膜拜吧！
 
详情可见酷壳原文：<http://coolshell.cn/articles/4883.html>
 
佩服之余，我在linux下用C语言实现了一把：

sleepsort.c

	#include <stdio.h>  
	#include <unistd.h>  
	#include <pthread.h>  
	void *f(void *opt)  
	{  
	    sleep((*((int *)opt)));  
	    printf("%d,",(*((int *)opt)));  
	    pthread_exit(0);  
	}  
	int main()  
	{  
	    const int len=5;  
	    pthread_t a_thread[5];  
	    int array[5]={3,5,2,4,1};  
	    int i=0;  
	  
	    while(i<len)  
	    {  
	    pthread_create(&(a_thread[i]),NULL,f,&(array[i]));  
	    i++;  
	    }  
	    i=0;  
	    while(i<len)  
	    {  
	    pthread_join(a_thread[i],0);  
	    i++;  
	    }  
	    printf("/n");  
	  
	    return 0;  
	}  

编译：

	gcc  -Wall  -o  sleepsort  sleepsort.c   -lpthread

执行：

	./sleepsort
	1,2,3,4,5,
