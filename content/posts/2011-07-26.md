---
title: Linux下分割、合并文件——dd和cat
date: 2011-07-26T08:00:00+08:00
draft: false
toc:
comments: true
---


dd的作用是转换和拷贝文件，我们可以利用它来分割文件，相关的选项如下：

* if=filename：输入的文件名

* of=finename：输出的文件名

* bs=bytes：一次读写的字节数，默认是512bytes

* skip=blocks:拷贝前，跳过的输入文件的前blocks块，块的大小有bs决定

* count=blocks：只拷贝输入文件的前blocks块
 
例如，现在有一个文件file，大小为116616字节：

	[root]# du -b file  
	116616  file  

将其分割为两文件file1和file2，那我们就设置每块为1024字节，将file的前60块放入file1，余下的放入file2：

	[root]# dd if=file bs=1024 count=60 skip=0  of=file1  
	[root]# dd if=file bs=1024 count=60 skip=60 of=file2  

然后用cat将两个文件合并为file.bak，要注意文件的顺序：

	[root]# cat file1 file2 > file.bak  

可以用md5sum验证一下file和file.bak：

	[root]# md5sum file  
	3ff53f7c30421ace632eefff36148a70  file  
	[root]# md5sum file.bak  
	3ff53f7c30421ace632eefff36148a70  file.bak  

可以证明两个文件时完全相同的。
 
为了方便分割、合并文件，我写了两个脚本：

ddf.sh

	#ddf.sh：分割文件，分割后的文件以数字结尾，例如file分割为两个文件：file1和file2  
	#!/bin/sh  
	  
	#使用脚本是第一参数是要分割的文件名  
	Filename=$1     
	Filesize=0  
	  
	Path=`pwd`  
	  
	#验证文件名是否正确，然后计算文件的大小  
	if [ -z $Filename ];then  
	    echo "Error:The file name can not be empty"  
	    exit  
	fi  
	if [ -e $Filename ];then  
	    Filesize=`du -b $Filename | awk '{print $1}'`  
	    if [ $Filesize == 0 ];then  
	        echo "Error:The File size is zero!"  
	        exit  
	    fi  
	    echo "The file size is $Filesize Byte"  
	    echo "Plese enter the subfile size(KB):"  
	else  
	    echo "Error:$Filename does not exist!"  
	    exit   
	fi  
	  
	#输入分割后每个文件的大小，单位是KB  
	read Subfilesize  
	if [ -z $Subfilesize ];then  
	    echo "Error:Input can not be empty"  
	    exit  
	fi  
	  
	echo $Subfilesize | grep '^[0-9]\+$' >> /dev/null  
	if [ $? -ne 0 ];then  
	    echo "Error:The Input is not a number!"  
	    exit  
	elif [ $Subfilesize -eq 0 ];then  
	    echo "Error:The Subfile size is zero!"  
	    exit  
	fi  
	  
	#计算需要分割为几个文件  
	SubfileByte=`expr $Subfilesize \* 1024`  
	Subfilenum=`expr $Filesize / $SubfileByte`  
	if [ `expr $Filesize % $Subfilesize` -ne 0 ];then  
	    Subfilenum=`expr $Subfilenum + 1`  
	fi  
	  
	#将文件分割  
	echo "$Filename will be divided into $Subfilenum"   
	i=1  
	skipnum=0  
	while [ $i -le $Subfilenum ]  
	do  
	    echo "$Filename$i"  
	    dd if=$Filename of="$Path/$Filename$i" bs=1024 count=$Subfilesize skip=$skipnum  
	    i=`expr $i + 1`  
	    skipnum=`expr $skipnum + $Subfilesize`  
	done  
	echo "$Filename has been divided into $Subfilenum"  
	echo "Done !"  

caf.sh

	#caf.sh:合并文件，需要合并的文件要放在一个文件夹里  
	#       文件名分为两个部分，第一部分都相同，第二部分必须是从1开始的连续数字，例如file1，file2，file3  
	#       合并后的文件名为file.bak  
	#!/bin/sh  
	  
	#输入文件名的第一部分  
	echo "Please enter file name:"  
	read Filename  
	if [ -z $Filename ];then  
	    echo "Error:File name can not be empty"  
	    exit  
	fi  
	  
	#输入待合并文件的个数  
	echo "Please enter the number of subfiles:"   
	read Subfilenum  
	if [ -z $Subfilenum ];then  
	    echo "Error:The number of subfiles can not be empty"  
	    exit  
	fi  
	echo $Subfilenum | grep '^[0-9]\+$' > /dev/null  
	if [ $? -ne 0 ];then  
	        echo "Error:Input must be a number"  
	exit  
	fi  
	if [ $Subfilenum -eq 0 ];then  
	        echo "Error:The number of subfiles can not be zero"  
	exit  
	fi  
	  
	  
	#合并文件  
	i=1  
	Newfile=$Filename\.bak  
	while [ $i -le $Subfilenum ]  
	do  
	    Subfilename=$Filename$i  
	        if [ -e $Subfilename ];then  
	            echo "$Subfilename done!"  
	            cat $Subfilename >> $Newfile  
	            i=`expr $i + 1`  
	        else  
	            echo "Error:$Subfilename does not exist"  
	            rm -rf $Newfile  
	            exit  
	        fi  
	done  
	  
	  
	echo "Subfiles be merged into $Newfile"  
	echo "Success!"  
 
用这两个脚本完成对file的分割、合并：

	[root]# ./ddf.sh file   
	The file size is 116616 Byte  
	Plese enter the subfile size(KB):  
	60  
	file will be divided into 2  
	file1  
	记录了60+0 的读入  
	记录了60+0 的写出  
	61440字节(61 kB)已复制，0.0352612 秒，1.7 MB/秒  
	file2  
	记录了53+1 的读入  
	记录了53+1 的写出  
	55176字节(55 kB)已复制，0.0316272 秒，1.7 MB/秒  
	file has been divided into 2  
	Done !  
	[root]# ls  
	caf.sh  ddf.sh  file  file1  file2  
	[root]# ./caf.sh   
	Please enter file name:  
	file  
	Please enter the number of subfiles:  
	2  
	file1 done!  
	file2 done!  
	Subfiles be merged into file.bak  
	Success!  
	[root]# ls  
	caf.sh  ddf.sh  file  file1  file2  file.bak
