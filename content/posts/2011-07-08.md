---
title: 关于C语言的位移操作
date: 2011-07-08T08:00:00+08:00
draft: false
toc:
comments: true
---


下面这段代码会输出什么结果？

	#include <stdio.h>  
	  
	int main()  
	{  
		printf("%d\n",( (i-sizeof(int)) >>32));  
		return 0;  
	}  

答案是 0

将代码修改如下：

	#include <stdio.h>  
	  
	int main()  
	{  
		printf("%d\n",( (i-((int)sizeof(int))) >>32));  
		return 0;  
	}  

结果输出 -1

解析：

sizeof是一个操作符，它返回的结果是unsigned int。

对于无符号数：右移后，高位补0；

对于有符号数：正数，右移后高位补0；负数，右移后高位补1。

建议只对无符号数做位运行，减少出错的可能。

注意：移动的位数要小于做操作数的总位数，否则结果是undefined（虽然有时也会得到正确的结果）。
