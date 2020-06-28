---
title: 关于函数的指针参数
date: 2011-07-01T08:00:00+08:00
draft: false
toc:
comments: true
---


先做一道题目：

	#include <stdio.h>  
	int f(int *p)  
	{  
		static int i = 5;  
		p=&i;  
		return 0;  
	}  
	int main(void)  
	{  
		int *p;  
		f(p);  
		printf("p = %d/n",*p);  
		return 0;  
	}  

该程序的输出结果是多少？  

有人会认为输出的是“p = 5”，其实应该是一个不确定的数。

为了弄清为什么，先看下一个例子：

	#include <stdio.h>  
	int main(void)  
	{  
		int i=0x01;  
		int *p=&i;  
		printf("&i = %x/n",&i);   //i的地址  
		printf("p = %x/n",p);  
		printf("*p = %x/n",*p);  
		printf("&p = %x/n",&p);  
		return 0;  
	}  

执行结果：

	&i = bfffe924
	p = bfffe924
	*p = 1
	&p = bfffe920
 
解析：

指针占4Byte，存放着它所指向的数据的地址。P是一个指向int型变量i的指针，所以p表示该指针的内容，也就是i的地址；\*p表示p指向的内容，就是i；&p代表指针p本身的地址。

再看第一道题，main函数里的p和f函数的参数p都是指向int型的指针，当 main 函数调用f函数时：f(p),传递给f函数的是main函数中的p的内容，因为p没有初始化，所以它的内容是一个不确定的地址，所以，f函数中的p指向了这个不确定地址。

怎样才能让 main 函数中的p指向f函数中的静态变量i呢？

将程序做如下修改即可：

	#include <stdio.h>  
	int f(int **p)  
	{  
		static int i = 5;  
		*p=&i;  
		return 0;  
	}  
	int main(void)  
	{  
		int i=0x01;  
		int *p=&i;  
		f(&p);  
		printf("p = %d/n",*p);  
		return 0;  
	}
