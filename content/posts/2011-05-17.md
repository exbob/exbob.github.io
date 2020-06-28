---
title: C语言的参数可变函数
date: 2011-05-17T08:00:00+08:00
draft: false
toc:
comments: true
---


环境：fedora12，gcc4.4.4
 
C语言参数个数可变的函数，叫做VA函数（variable argument function），例如printf()函数。要实现VA函数需要包含stdarg.h:

	#include <stdarg.h>

主要使用下面三个宏定义：

	va_start(ap, A)
	va_arg(ap, T)
	va_end(ap)
 
ap 是类型是 va\_list ，va\_list 的定义如下：

	typedef char *va_list;

它用来指向函数的参数。
 
A是函数的最后一个固定参数，一个VA函数必须有至少一个固定参数。
 
T是参数的类型，例如int，char...,等等。
 
* va_start(ap,A)：初始化函数参数指针ap，使ap指向A的右边的第一个参数，即函数可变参数中的第一个；
* va_arg(ap,T）：返回ap指向的参数的值，参数的类型用T确定，然后将ap指向下一个参数；
* va_end（ap）：释放ap。
 
下面的例子定义了一个va函数，作用是依次打印传递给函数的int型参数。

	#include <stdio.h>
	#include <stdarg.h>
	
	int va_function(const char *start,...)
	{
		int n=1;
	
		va_list arg;
		va_start(arg,start);
		printf("%s/n",start);
	
		while(n != 0)
		{
			n=va_arg(arg,int);
			printf("%d/n",n);
		}
	
		va_end(arg);
	
		return 0;
	}
	
	
	int main(void)
	{
		va_function("start",1,2,3,-1,-2,0);
	         return 0;
	}


注意：

1. VA函数的可变参数类型必须在函数中自行确定，编译器无法自动识别。
2. 必须在VA函数中定义可变参数的结束标志，如果没有结束标志，va_arg会依次返回内存中的值，直到访问到非法内存而出错退出。

这几个宏在不同的编译器中有不同的定义，但是功能是一样的，在linux的内核源码中也用定义，可以参考。在内核源码的 `/include/acpi/platform/acenv.h` 文件内：

	typedef char *va_list;
	　　
	/*
	Storage alignment properties 
	*/
	#define _AUPBND (sizeof (acpi_native_uint) - 1)
	#define _ADNBND (sizeof (acpi_native_uint) - 1)
	　
	/*
	Variable argument list macro definitions 
	*/
	#define _bnd(X, bnd) (((sizeof (X)) + (bnd)) & (~(bnd)))
	#define va_arg(ap, T) (*(T *)(((ap) += (_bnd (T, _AUPBND))) - (_bnd (T,_ADNBND))))
	#define va_end(ap) (void) 0
	#define va_start(ap, A) (void) ((ap) = (((char *) &(A)) + (_bnd (A,_AUPBND))))

参考：

[深入浅出VA函数](http://writeblog.csdn.net/#resources)

[linux kernel中的变长参数宏](http://blog.csdn.net/storylike/archive/2011/01/26/6164240.aspx)

《The Open Group Base Specifications Issue 6》
