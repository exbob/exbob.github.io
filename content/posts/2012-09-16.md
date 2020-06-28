---
title: Sublime Text
date: 2012-09-16T08:00:00+08:00
draft: false
toc:
comments: true
---


## 简介
官方 : <http://www.sublimetext.com/>  

参考了非官方文档 : <http://docs.sublimetext.info/en/latest/index.html>  

视频简介 : <http://v.youku.com/v_show/id_XMzU5NzQ5ODgw.html>

## 1.基本编辑

### 1.1.操作文件
* Ctrl+n :新建文件。
* Ctrl+s :保存文件。
* Ctrl+w :关闭当前文件，没有文件时会关闭窗口。
* Ctrl+tab :在多个打开的文件之间切换
* 在 View->Syntax 中可以选择当前文件的语法。

### 1.2.选择文本
* Ctrl+l :选择当前行
* Shift+鼠标右键 :按列选择文本。
* 反复按下 Ctrl+d 可以将全文中与光标所在处相同的单词逐一加入选择，直接按 Alt+F3 可以一次选择全部相同的单词。

### 1.3.跳转
* 用 Ctrl+P 可以快速跳转到当前项目中的任意文件，可进行关键词匹配。
* 用 Ctrl+P 后 @ (或是Ctrl+R)可以快速列出/跳转到某个函数（很爽的是在 markdown 当中是匹配到标题，而且还是带缩进的！）。
* 用 Ctrl+P 后 # 可以在当前文件中进行搜索。
* 用 Ctrl+P 后 : (或是Ctrl+G)加上数字可以跳转到相应的行。
* 而更酷的是你可以用 Ctrl+P 加上一些关键词跳转到某个文件同时加上 @ 来列出/跳转到目标文件中的某个函数，或是同时加上 # 来在目标文件中进行搜索，或是同时加上 : 和数字来跳转到目标文件中相应的行。

<!-- more -->

### 1.4.注释
* Ctrl+/ :快速注释一行，会根据当前文件的语法选择注释字符。
* Ctrl+Shift+/ :注释一块代码。


## 2.常用设置
Sublime Text 的各项特性都是用配置文件来设置的。例如用 Preferences->Settings-User 可打开针对当前用户的配置文件，文件中每行设置一个特性，以逗号结尾，最后一个行末尾不可以写逗号，例如:

	{
		"font_face": "DejaVu Sans Mono",
		"font_size": 12
	}

### 2.1.设置字体
点击菜单上的 Preferences->Settings-User ，在打开的用户配置文件中输入:

	"font_face": "DejaVu Sans Mono",
	"font_size": 12,

也可以选择 Preferences->Settings-Default，在默认配置文件中的相同字段中设置默认字体。

### 2.2.设置缩进
在 Preferences->Settings-Default 中可以看到默认缩进为4个空格的宽度，并且 Tab 不会转换为空格，相应的字段为:

	"tab_size": 4,
	"translate_tabs_to_spaces": false,

还可以针对相应的语法设置特定的缩进格式。先在 View->Syntax 中选择语法，然后点击 Preferences->Settings-More->Syntax Specific-User,在打开的文件中设置。

### 2.3.设置文件的保存格式
我们希望根据当前的语法将文件保存为相应的格式，例如Markdown文件保存为后缀为 .md 的文件。设置方法如下:

1. 新建一个文件，设置 View->Syntax 为 Markdown 。

2. 选择 preferences->Setting-more->Syntax Specific-User ，在打开的 Markdown.sublime-settings 文件中输入:

		{
			"extensions": [ "md" ] 
		}  

	然后保存。

3. 保存之前新建的文件，后缀就会是 .md 。

### 2.4.设置为Vim模式
点击 Preferences->Setting-user ，在打开的配置文件中添加：

	"ignored_packages": [],

然后保存。

在 Vim 的 insert 模式下也可以用正常 Sublime Text 的方式编辑文件。

## 3.安装插件
只需将下载的插件解压到 Packages 目录即可，打开 Packages 目录的方法是点击 Preferences->Browse Packages... 。

也可以通过 Sublime Package Control 插件来搜索、下载、安装源中的插件，方法如下：

1. 用 Ctrl+· 打开 console ，输入如下命令:

		import urllib2,os; pf='Package Control.sublime-package'; ipp=sublime.installed_packages_path(); os.makedirs(ipp) if not os.path.exists(ipp) else None; urllib2.install_opener(urllib2.build_opener(urllib2.ProxyHandler())); open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read()); print 'Please restart Sublime Text to finish installation'

	该命令会创建一个 Installed Packages Text 文件夹，然后下载 Package Control.sublime-package 到他下面。

2. 然后重启 Sublime Text 。在菜单 Preferences 下可以看到多出了 Package Control 条目。

3. 按下 Ctrl+Shift+p ，打开命令面板。输入 install 调出 Package Control: Install Package 后按回车，在列表中选择需要安装的插件，就会自动安装，重启后生效。

### 3.1.一些有用的插件
* GBK Encoding Support : Sublime Text 目前只支持 UTF-8 编码的文件，该插件可以使 Sublime Text 支持 GBK 编码文件。启用后载入文件的速度变慢，还有一个Bug就是标签上的文件名乱码，根据某网友的方法对该插件的 sublime_gbk.py 文件的进行修改后解决，修改后的代码如下，用 # 注释的是源代码，用 #new line 标记的是新加的代码。

		except:
			gbk = file(view.file_name()).read()
			text = gbk.decode('gbk')
			
			file_name = view.file_name().encode('utf-8')
	
			#tmp_file_name = urllib.quote_plus(os.path.basename(file_name))  + SEPERATOR + urllib.quote_	plus(file_name)
			#tmp_file = os.path.join(TEMP_PATH, tmp_file_name)
	
			#f = file(tmp_file, 'w')
			f = file(view.file_name(),'w')  #new line
			f.write(text.encode('utf8'))
			f.close()
	
			window = sublime.active_window()
			
			#v = window.find_open_file(tmp_file)
			v = window.find_open_file(file_name)  #new line
	
			if(not v):
				#window.open_file(tmp_file)
				window.open_file(file_name)  #new

* ConvertToUTF8 : 功能与 GBK Encoding Support 类似，且没有 Bug 。

* Markdown Build : 用于将 Markdown 文件转换为 Html ，并用浏览器打开。快捷键是 Ctlr+b 。

* Tag : 格式化 Html 和 CSS 语句，选中一段语句，安装组合键 ctrl+alt+f 即可。

* JSFormat : 格式化 Javascript 文件

	安装后重启 sublime text 就能使用js格式化插件
	
	使用方法：
	
	1.快捷键：ctrl+alt+f
	
	2.先用快捷键打开命令面板 “ctrl + shift + p”, 再输入 “Format: Javascript” 就可以使用格式化命令

* SublimeCodeIntel : 代码补全提示，支持多种语言。

* Soda : 一个主题，比默认的漂亮一些。安装后，打开配置文件Preferences -> Settings - User，加上一行"theme": "Soda Light.sublime-theme"或者 "theme": "Soda Dark.sublime-theme"。前面一个是亮色主题，后面一个是暗色主题。


### 3.2.用 Sublime Text 编辑 Markdown 文件
1. 配置 Markdown 语法高亮

	下载 [Made of code](https://github.com/kumarnitin/made-of-code-tmbundle) 配色文件，放在 Preferences->Browse Package... 所打开的目录下即可。

	在 Preferences->Color Scheme 中选择 Made of code ，在 View->Syntax 中选择 Markdown 。

2. 安装 Markdown Build 插件。

3. 在 Tool 菜单里点击 Build，快捷键是 Ctrl+b ，会调用 Markdown Build 生成 Html 文件并用默认的浏览器打开。

## 4.一些问题

最近发现打开文件时，会多出一个带 dump 后缀的文件。最后发现是 GBK Encoding Support 插件的问题，卸载后解决，然后换上了 ConvertToUTF8 。
