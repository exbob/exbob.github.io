---
title: 用 Octopress 在 github 上部署博客
date: 2012-09-09T08:00:00+08:00
draft: false
toc:
comments: true
---


## 1.准备

* 安装 git
	
		sudo yum install git

	配置：

		git config --global user.name "your name"
		git config --global user.email "your email"

* 用 RVM 安装 Ruby 1.9.3：

		curl -L https://get.rvm.io | bash -s stable --ruby

	根据提示执行：

		source ~/.rvm/scripts/rvm

	再安装 ruby 1.9.3：

		rvm install 1.9.3
		rvm use 1.9.3
		rvm rubygems latest

	安装结束后运行 ruby --version 查看 ruby 版本。

<!-- more -->

* 安装 Octopress

		git clone git://github.com/imathis/octopress.git octopress
		cd octopress    # 首次进入该目录时会询问是否信任 .rvmrc 文件，选择 yes 
		ruby --version  # 应该是 Ruby 1.9.3

	再安装一些依赖文件：

		gem install bundler
		bundle install

	安装默认的主题：

		rake install

## 2.部署到 Github Pages

先在 github 上新建一个名为 username.github.com (username 替换为你的用户名) 的版本库。然后执行：
	
	rake setup_github_pages

这里会询问你的 Github Pages 版本库的 URL ，应该输入 git@github.com:username/username.github.com.git 。这条命令还会做以下几件事：

* 将 Github Pages 版本库作为默认的 origin remote
* 将活动分支从 master 切换到 source 
* 根据版本库设置博客的 URL
* 在 _deploy 目录下设置一个 master 分支用于部署博客。

然后执行：

	rake generate
	rake deploy

会在 _deploy 目录下生成博客，并把该目录下的文件发布到 github 版本库的 master 分支。十分钟以后就可以通过 username.github.com 访问你的博客了。

Octopress 的源文件也应该提交到 github 保存：

	git add .
	git commit
	git push origin source

## 3.配置

Octopress 的配置文件有四个：
	
	_config.yml   #主配置文件（jekyll的设置）
	Rakefile      #关于部署的配置
	config.rb
	config.ru

通常只需要修改 _config.yml 和 Rakefile 。

_config.yml 文件中的配置分为三个部分：

**Main Configs**
主要的设置有：

	url:                # 用于 RSS 的 URL
	title:              # 博客的标题，会显示在博客和浏览器标签上。
	subtitle:           # 博客的副标题
	author:             # 名字, 用于 RSS, Copyright, Metadata
	simple_search:      # 搜素栏所用的网址
	description:        # 默认的关于本站点的描述
	date_format:        # Format dates using Ruby's date strftime syntax
	subscribe_rss:      # 用于 rss 订阅的 URL, 默认用 /atom.xml
	subscribe_email:    # Url to subscribe by email (service required)
	category_feeds:     # Enable per category RSS feeds (defaults to false in 2.1)
	email:              # Email address for the RSS feed if you want it.

**Jekyll & Plugin**

用于 Jekyll 及其插件的设置，主要有：

	root:               # 相对根目录，默认是 / ，如果博客要发布在一个子目录下（例如 site.com/project），要设为 /project 
	permalink:          # 博客文章的存放结构
	source:             # 源文件的目录
	destination:        # 生成的文件存放的目录
	plugins:            # Jekyll 插件的目录
	code_dir:           # 代码片段文件的存放目录 (用于 include_code 插件)
	category_dir:       # Directory for generated blog category pages
	
	pygments:           # Toggle python pygments syntax highlighting
	paginate:           # Posts per page on the blog index
	pagination_dir:     # Directory base for pagination URLs eg. /blog/page/2/
	recent_posts:       # Number of recent posts to appear in the sidebar
	
	default_asides:     # Configure what shows up in the sidebar and in what order
	blog_index_asides:  # Optional sidebar config for blog index page
	post_asides:        # Optional sidebar config for post layout
	page_asides:        # Optional sidebar config for page layout

**3rd Party Setting**

这些第三方工具已经安装，只要简单的设置就可以使用。


## 4.写博客

**发表文章**

所有要发表的博文都存放在 source/_posts 目录下，并遵循 Jekyll 的命名方式：YYYY-MM-DD-post-title.markdown 。可以将写好的文章按照这种方式命名，并在文件中添加下面讲到的头部，然后复制到该目录习下。也可以用 Octopress 提供的命令生成，语法如下：

	rake new_post["tiltle"]

这样生成的文件默认是 markdown 格式，可以在 Rakefile 中修改默认的格式。

打开文件会发现有一个头部，类似：
	
	---
	layout: post
	title: "tiltle"
	date: 2012-09-03 15:59
	comments: true
	external-url:
	categories:
	---

layout: post 表示使用 source/_layouts/post.html 布局方式。 comment: true 表示允许评论，也可设为 off 关闭评论。如果有其他作者，可以添加 author: your name ，默认会使用 _config.yml 中的设置。如果要保存为草稿，添加 published: false ，这样在生成博客时就不会被发表。categories 用于设置标签，如果只有一个标签，可以用如下格式：

	categories: Sass

如果有多个标签：

	# Multiple categories example 1
	categories: [CSS3, Sass, Media Queries]
	
	# Multiple categories example 2
	categories:
	- CSS3
	- Sass
	- Media Queries

默认情况下，post 的全部内容将显示在 index 页面。可以在文章中插入 &lt;!-- more --> ，index 页面只会显示该标记之前的内容，点击 “Continue->” 按钮后会完全显示。

**添加页面**

可以在 source 目录下添加页面，添加的文件的路径与最终生成的博客的 URL 是直接对应的，例如 about.markdown 就对应 site.com/about.html 。如果想要 site.com/about ，就要在 source 下添加 about/index.markdown 。可以用 rake 命令添加页面：

	rake new_page[super-awesome]
	# 生成 source/super-awesome/index.markdown

	rake new_page[super-awesome/page.html]
	# 生成 source/super-awesome/page.html

默认添加的文件的格式是 markdown ，也可以在 Rakefile 中修改。文件的内容与 post 类似：

	---
	layout: page
	title: "Super Awesome"
	date: 2011-07-03 5:59
	comments: true
	sharing: true
	footer: true
	---

**生成和预览**

执行：

	rake generate   

在 _site 目录下生成博客的静态页面。

然后执行：

	rake preview    # Watches, and mounts a webserver at 

在浏览器中访问 http://localhost:4000 即可预览博客。

最后用 rake deploy 发布。
