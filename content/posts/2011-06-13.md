---
title: Unicode编码字符的点阵显示
date: 2011-06-13T08:00:00+08:00
draft: false
toc:
comments: true
---


对于unicode编码的字符，可以先将unicode编码转换为GBK编码，然后利用GBK点阵字库（gbk.bin)显示字符。一个比较好的编码转换方法是，制作一个按照unicode编码顺序排列的GBK编码表，直接根据unicode编码就可以查到对应的GBK编码，具体的制作方法如下：
 
## 1.生成一个unicode字符文件

上以篇文章中已经制作了一个GBK编码的全字符文件gbk.txt，用记事本打开gbk.txt，然后以unicode编码另存为unicode.txt文件。

这样的话，所有的GBK编码字符都用unicode编码保存在unicode.txt中了，可以用winhex打开unicode.txt，会发现前两个字节是FFFE，这是unicode编码的标识，要注意后面的unicode编码是小端存储。
 
## 2.制作unicode转gbk的文件

这里要用程序制作一个unicode与gbk编码相对应的文件，该文件每四个字节为一组，其中前两个字节为unicode编码，后两个字节是对应的gbk编码，两种编码都以小端存储，数据结构如下：

	union code
	
	{
	
	unsigned int unigbk;
	
	unsigned short int uni_gbk[2];
	
	unsigned char buffer[4];
	
	};

这个数据结构是以gbk编码的顺序存放在文件uni2gbk.txt中，文件大小为128764Byte。程序如下：

	//由unicode.txt和gbk.txt生成uni2gbk.txt
	#include <stdio.h>
	
	int main(void)
	{
		unsigned char buffer[2];
		int count=0;
	
		FILE *fup=0;
		FILE *fgp=0;
		FILE *fwp=0;
	
		fup=fopen("unicode.txt","rb");
		fgp=fopen("gbk.txt","rb");
		fwp=fopen("uni2gbk.txt","wb");
	
		fgetc(fup);
		fgetc(fup);
	
		while(1)
		{
			//读取unicode编码，写入uni2gbk.txt
			buffer[0]=fgetc(fup);
			buffer[1]=fgetc(fup);
			fputc(buffer[0],fwp);
			fputc(buffer[1],fwp);
			//读取gbk编码，写入uni2gbk.txt
			buffer[0]=fgetc(fgp);
			buffer[1]=fgetc(fgp);
			fputc(buffer[1],fwp);
			fputc(buffer[0],fwp);
	
			count+=2;
			if(count==0xfb7c)
				printf("xxxxxxxxx/n");
	
	
			if(buffer[0]==0xfe && buffer[1]==0xfe)
			{
				printf("count = %x/n",count);
	
				fclose(fup);
				fclose(fgp);
				fclose(fwp);
	
				return;
			}
		}
		return 0；
	}

## 3.将uni2gbk.txt排序

用程序将uni2gbk.txt文件中的union code数据结构按照unicode编码的顺序从小到大排序，程序如下：

	#include <stdio.h>
	#include <stdlib.h>
	
	#define MAXSIZE (0x7DBF)  //union code的个数
	
	union code
	{
		unsigned int unigbk;
		unsigned short int uni_gbk[2];
		unsigned char buffer[4];
	};
	
	
	int main(void)
	{
		unsigned char flag=1;
	
		unsigned int i=0;
		unsigned int j=0;
	
		union code pdata;
		union code ndata;
	
		FILE *fp=0;
	
		fp = fopen("uni2gbk.txt","rb+");
	
		for(i=1; i<MAXSIZE && flag==1; i++)
		{
			flag=0;
	
			for(j=0; j<(MAXSIZE-i); j++)
			{
				fseek(fp,j*4,0);
				fread(pdata.buffer,1,4,fp);
				fseek(fp,j*4+4,0);
				fread(ndata.buffer,1,4,fp);
	
				if(pdata.uni_gbk[0]>ndata.uni_gbk[0])
				{
					flag=1;
	
					fseek(fp,j*4,0);
					fwrite(ndata.buffer,4,1,fp);
					fseek(fp,j*4+4,0);
					fwrite(pdata.buffer,4,1,fp);
	
				}
	
			}
	
			printf("i=%d/n",i);
		}
	
		fclose(fp);
	
	}

排序后，为了与没有排序的uni2gbk.txt区分，将文件名改为uni2gbkp.txt。
 
## 4.删除无用的编码

用winhex打开uni2gbkp.txt，可以发现，从0到第0x80EB个字节中的unicode编码都是0x0020或0x003F，这些都是无用的编码，它们所对应的gbk编码也是无用的，都可以删除。

删除后，uni2gbkp.txt文件中的union code数据结构的unicode编码就是从0x00A4开始，以0xFFE5结束。但是这些unicode编码不是连续的，例如0x00A4之后就是0x00A7，为了方便查找，需要在不连续的编码中间用union code填充，对应的gbk编码部分用0x0000填充，然后将union code中的unicode编码全部删除。最后生成uni2gbk.bin文件。程序如下：
 
	#include <stdio.h>
	#include <stdlib.h>
	
	union code
	{
		unsigned int unigbk;
		unsigned short int uni_gbk[2];
		unsigned char buffer[4];
	};
	
	int main()
	{
		unsigned short int count=0x00A4;  //从unicode的0x00A4开始
		union code temp;
	
		FILE *frp=0;
		FILE *fwp=0;
	
		frp=fopen("uni2gbkp.txt","rb");
		fwp=fopen("uni2gbk.bin","wb");
	
		fseek(frp,0x80EC,SEEK_SET);  //舍弃uni2gbkp.txt文件的前0x80EC个字节
		fread(temp.buffer,1,4,frp);
	
		while(count<=0xffe5) //以unicode的0xffe5结束
		{
			if(temp.uni_gbk[0]==count)     //判断unicode编码是否连续
			{
				fputc(temp.buffer[2],fwp);  //将对应的gbk编码写入uni2gbk.bin
				fputc(temp.buffer[3],fwp);
				fread(temp.buffer,1,4,frp);
			}
			else                        //不连续的地方填充0
			{
				fputc(0x00,fwp);
				fputc(0x00,fwp);
			}
	
			count++;
		}
		fclose(frp);
		fclose(fwp);
		return 0;
	}

生成的uni2gbk.bin文件中都是gbk编码，这些gbk编码都是按照对应的unicode编码（0x00A4到0xFFE5）的顺序排列的。
 
## 5.uni2gbk.bin的使用

假设一个字符的unicode编码为X，那么它对应的gbk编码在uni2gbk.bin文件中的位置就是：

（X-0x00A4）*2

从这个位置读取一个unsigned short int数据，就是它的gbk码。然后利用GBK点阵字库（gbk.bin）即可显示。

例如，以Linux的终端模拟点阵屏幕，程序如下：

	#include <stdio.h>
	#include <unistd.h>
	#include <curses.h>
	
	#define START 0x8140
	#define DATANUM 0x20
	
	int displaychar(FILE *fp,unsigned short int dispch,char fillch,char start_x,char start_y);
	
	int main(void)
	{
		FILE * fp=0;
		unsigned short int testch = 0x7231;  //汉字'爱‘的unicode码
		unsigned short int gbkch=0;       //存放testch的gbk编码
	
		//将unicode转换为gbk
		fp = fopen("uni2gbk.bin","rb");
		fseek(fp,(testch-0x00A4)*((unsigned int)2),SEEK_SET);
	
		gbkch = fgetc(fp);
		gbkch = (fgetc(fp)<<8) + gbkch;
	
		fclose(fp);
	
		fp = fopen("gbk.bin","rb");
	
		initscr();
	
		displaychar(fp,gbkch,'*',0,0);
	
		refresh();
	
		while(1);
		endwin();
	
		fclose(fp);
		return 0;
	}
	
	/*
	 * fp指向点阵字库二进制文件
	 * 以点阵方式显示一个GBK字符
	 * dispch是要显示的字符，fillch是填充点阵的字符
	 * start_x,start_y是显示的起始坐标
	 */
	int displaychar(FILE *fp,unsigned short int dispch,char fillch,char start_x,char start_y)
	{
		char x=start_x;
		char y=start_y;
		unsigned int location=(dispch-START)*DATANUM;
	
		int i=0;
		int j=0;
		char buf=0;
	
		fseek(fp,location,SEEK_SET);
	
		for(i=0;i<DATANUM;i++)
		{
			buf=fgetc(fp);
	
			//显示一个字节
			for(j=0;j<8;j++)
			{
				move(y+j,x);
				if( buf & (0x01<<j) )
				{
					addch(fillch);
				}
			}
	
			if(x == (start_x+15))
			{
				x=start_x;
				y=start_y+8;
			}
			else
				x++;
		}
		return 0;
	
	}

## 下载：

unicode点阵字库文件：<http://download.csdn.net/source/3362591>
