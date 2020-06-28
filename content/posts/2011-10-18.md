---
title: 不用任何变量，实现strlen函数
date: 2011-10-18T08:00:00+08:00
draft: false
toc:
comments: true
---


使用递归：

	int strlen(char * str)  
	{  
	     if(*str)            
	          return strlen(str+1)+1;  
	     else  
	          return 0;  
	}
