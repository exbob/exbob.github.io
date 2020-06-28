---
title: 用getopt处理main函数的参数
date: 2011-05-28T08:00:00+08:00
draft: false
toc:
comments: true
---


环境：Fedora12  Gcc4.4.2
 
在C语言中，main函数的声明如下：

	int main（int argc，char *argv[]);

argc是指程序参数的个数，包括程序名本身，如果程序不带参数，argc为1；

argv的每个数组元素存放一个程序参数，程序名存放在argv[0];

例如：

	$ ls -l

此时，argc为2，argv[0]是ls，argv[1]是-l。
 
 
程序的参数可以分为三种：选项，选项的关联值，非选项参数。例如：

	$gcc hello.c -o hello  

hello.c是非选项参数，-o是选项，hello是-o选项的关联值。

根据Linux的惯例，程序的选项应该以一个短横线开头，后面包含单个字母或数字，选项分为两种：带关联值的和不带关联值的，例如：
 
	$gcc hello.c -o hello

选项-o必须带一个关联值。

	$ls -l

选项-l无需带参数。

不带关联值的选项应该可以在一个短横线后合并使用，例如：

	$ls -la
 
Linux系统提供了getopt函数，它用来按照上述规则处理程序的参数，相关定义如下：

	#include <unistd.h>
	 
	int getopt(int argc,char *const argv[],const char *optstring);
	extern char *optarg;
	extern int optind,opterr,optopt;
 
getopt 使用 main 函数的 argc 和 argv 作为前两个参数，optsting 是一个字符列表，每个字符代表一个单字符选项，如果一个字符后面紧跟以冒号（：），表示该字符有一个关联值作为下一个参数；
 
getopt的返回值是argv数组中的下一个选项参数，由optind记录argv数组的下标,如果选项参数处理完毕，函数返回-1；

如果遇到一个无法识别的选项，返回问号（？），并保存在optopt中；

如果一个选项需要一个关联值，而程序执行时没有提供，返回一个问号（？）,如果将optstring的第一个字符设为冒号（:),这种情况下，函数会返回冒号而不是问号。
 
选项参数处理完毕后，optind 会指向 argv 数组尾部的其他非选项参数。实际上，getopt 在执行过程中会重排 argv 数组，将非选项参数移到数组的尾部。
 
例：

下面这个程序需要提供两个无关联值的选项：-v，-g；一个需要关联值的选项：-t；一个非选项参数。

	//getopttest.c
	#include <stdio.h>
	#include <unistd.h>
	
	int main(int argc,char *argv[])
	{
	
		int opt=0;
		int i=0;
	    const char *optstring=":vgt:";
	    //非选项参数的个数
	    const int num=1;
	    //调用getopt前打印argv数组
	    for(i=0;i<argc;i++)
	    	printf("%d:%s/n",i,argv[i]);
	
	    //处理选项参数
	    while((opt=getopt(argc,argv,optstring)) != -1)
	    {
			switch(opt)
			{
	
			case 'v':
			case 'g':
				printf("option：%c/n",opt);
				break;
			case 't':
				printf("option:%c = %s/n",opt,optarg);
				break;
			case ':':
				printf("the option need a value/n");
				break;
			case '?':
				printf("unknow option：%c/n",optopt);
				break;
	    	}
	    }
	
	    //处理非选项参数
	    for(i=0;optind<argc;i++,optind++)
	    {
	    	if(i<num)
	    		printf("argument:%s/n",argv[optind]);
	    	else
	    		printf("excess argument:%s/n",argv[optind]);
	    }
	
	    //调用getopt后打印argv数组
	    for(i=0;i<argc;i++)
	    	printf("%d:%s/n",i,argv[i]);
	
	    return 0;
	}

编译：

	gcc -Wall getopttest.c -o getopttest

执行：

	$./getopttest arg1 -vg -t value -x arg2
	0:./getopttest
	1:arg1
	2:-vg
	3:-t
	4:value
	5:-x
	6:arg2
	option：v
	option：g
	option:t = value
	unknow option：x
	argument:arg1
	excess argument:arg2
	0:./getopttest
	1:-vg
	2:-t
	3:value
	4:-x
	5:arg1
	6:arg2
 
从执行结果可以看出，getopt 函数重排了 argv 数组，将非选项参数 arg1 排到了数组尾部。
