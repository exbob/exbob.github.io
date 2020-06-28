---
title: Octopress 使用笔记
date: 2012-09-15T08:00:00+08:00
draft: false
toc:
comments: true
---


## 1.代码和语法高亮

除了 Markdown 语法的缩进方式，在 Octopress 中还多种方式可以在文章中嵌入代码并显示语法高亮：

* 反引号代码块

语法

	``` [language] [title] [url] [link text]
	code snippet
	```

* 嵌入 Gist

[Gist](https://gist.github.com/) 是 Github 提供的一个代码管理功能。在 Gist 中新建的代码文件会获得一个 gist_id ，用下面的语法就可以在文章中嵌入相应的代码：

	{ % gist gist_id [filename] % }

filename 是可选的，如果一个 gist_id 下有多个文件，可以依次嵌入。

* include_code
	
include_code 用于在文章中导入代码文件。可以在 _config.yml 的 code_dir 设置文件路径，默认是 source/downloads/code 。

语法：

	{ % include_code [title] [lang:language] path/to/file % }

例如，导入 source/downloads/code/test.js ：

	{ % include_code test.js % }

<!-- more -->

## 2.插入图片

github 提供的空间有限，图片最好使用外链。

Octopress 提供的插入图片的语法如下：

	{ % img [class names] /path/to/image [width] [height] [title text [alt text]] % }

如果只输入图片的路径 (/path/to/image) ，图片会以左对齐的方式显示，其他可选参数可以控制图片的宽高和显示位置 (class names) 。

## 3.添加导航

以添加一个 About 页面为例：

* 执行：
		rake new_page[blog/about]
	在 source/blog/ 目录先生成 about 目录和 about/index.markdown 文件，默认的布局方式是 layout: page 。

* 在 source/_includes/custom/navigation.html 文件中添加一行：

``` html

		<li><a href="{{ root_url }}/blog/about">About</a></li>

```

## 4.布局

所有的页面布局文件都位于 source/_layouts 目录下，例如 post.html 是发布文章时默认的布局。

可以从已有的布局文件复制，然后修改，例如从 page.html 复制一份 about.html ，并修改，然后就可以在上面新建的 about/index.markdown 中设置 layout: about 。

## 5.侧边栏

Octopress 提供的侧边栏都位于 source/_includes/asides 目录下，例如：显示最近发表的 recent_posts.html ，显示 github 版本库的 github.html。我们新增的侧边栏最好放在 source/_includes/custom/asides 下。

假设我们新建了一个显示发布许可证的侧边栏:license.html，要将其显示在所有的页面需要修改 _config.yml 文件：

	default_asides: [custom/asides/license.html]

default_asides 中添加的侧边栏会显示在所有布局的页面中，如果想要根据页面的布局显示不同的侧边栏，Octopress 提供了三个默认布局的设置变量：

	blog_index_asides # blog 主页上显示的侧边栏
	post_asides       # post 布局页面显示的侧边栏
	page_asides       # page 布局页面显示的侧边栏

只要在相应的变量中添加侧边栏即可。

如果要新建了其他布局，如 about.html，只需在 _config.yml 文件中添加 about_asides 。

## 6.评论

Octproess 已经包含了支持 [Disqus](http://disqus.com) 的插件，只需做简单的设置即可使用：

* 首先确保你已经注册了一个 Disqus 的账户，并将你的 Blog 的 URL 添加到 Disqus 。
* 然后在 _config.yml 中设置 Disqus 账户的 short name , 并将其使能：

		# Disqus Comments
		disqus_short_name: exbob
		disqus_show_comment_count: true

* 最后在每个 post 的头部设置允许评论：
		
		comments: true

## 7.分享

添加分享按钮的方式有很多，这里用的是 [百度分享](share.baidu.com)：

* 首先在百度分享中选择分享按钮的样式，并生成代码，例如：

``` html

<!-- Baidu Button BEGIN -->
<div id="bdshare" class="bdshare_t bds_tools get-codes-bdshare">
	<a class="bds_qzone"></a>
	<a class="bds_tsina"></a>
	<a class="bds_tqq"></a>
	<a class="bds_renren"></a>
	<span class="bds_more">更多</span>
	<a class="shareCount"></a>
</div>
<script type="text/javascript" id="bdshare_js" data="type=tools&amp;uid=949520" ></script>
<script type="text/javascript" id="bdshell_js"></script>
<script type="text/javascript">
	document.getElementById("bdshell_js").src = "http://bdimg.share.baidu.com/static/js/shell_v2.js?cdnversion=" + new Date().getHours();
</script>
<!-- Baidu Button END -->

```
* 将上面的代码添加到 source/_includes/post/sharing.html 文件的中：

``` html

<div class="sharing">

{ % if site.baidushare % }

<!-- Baidu Button BEGIN -->
<div id="bdshare" class="bdshare_t bds_tools get-codes-bdshare">
	<a class="bds_qzone"></a>
	<a class="bds_tsina"></a>
	<a class="bds_tqq"></a>
	<a class="bds_renren"></a>
	<span class="bds_more">更多</span>
	<a class="shareCount"></a>
</div>
<script type="text/javascript" id="bdshare_js" data="type=tools&amp;uid=949520" ></script>
<script type="text/javascript" id="bdshell_js"></script>
<script type="text/javascript">
	document.getElementById("bdshell_js").src = "http://bdimg.share.baidu.com/static/js/shell_v2.js?cdnversion=" + new Date().getHours();
</script>
<!-- Baidu Button END -->
{ % endif % }

</div>

```

* 最后，在 _config.yml 文件中添加 baidushare :

```
		# baidu share
		baidushare: true
```
