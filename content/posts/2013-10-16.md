---
title: Google Fonts
date: 2013-10-16T08:00:00+08:00
draft: false
toc:
comments: true
---


Google Fonts 是一套高品质的 Web 字体，可以通过 Google Fonts API 添加到任意 Web 页面中，只需在 HTML 文件的头部添加一个 `stylesheet` 链接，然后就可以通过 CSS 使用字体。

优势：

* 高品质，开源
* 支持大多数浏览器
* 极其易用

劣势：

* 墙内链接不稳定

Google Fonts API ：<https://developers.google.com/fonts/>

## Example

下面是一个简单的例子，将这份代码复制并保存为 html 文件。

	<html>
		<head>
			<link rel="stylesheet" type="text/css" href="https://fonts.googleapis.com/css?family=Tangerine">
			<style>
				body {
					font-family: 'Tangerine', serif;
					font-size: 48px;
				}
			</style>
		</head>
		<body>
			<div>Making the Web Beautiful!</div>
		</body>
	</html>


然后在浏览器中打开这个 html 文件，就会看到如下效果：

<p style="font-family:'Tangerine';font-size:48px">Making the Web Beautiful!</p>

这是普通文本，所以可以用 CSS 改变它的样式，试着添加阴影:

	body {
		font-family: 'Tangerine', serif;
		font-size: 48px;
		text-shadow: 4px 4px 4px #aaa;
	}

效果如下：

<p style="font-family:'Tangerine';font-size:48px;text-shadow: 4px 4px 4px #aaa">Making the Web Beautiful!</p>

## Overview

使用 Google Fonts API 只需两步：

1. 添加一个 stylesheet 链接：

		<link rel="stylesheet" type="text/css" href="https://fonts.googleapis.com/css?family=Font+Name">

2. 为一个元素指定需要的字体：

		CSS selector {
		  font-family: 'Font Name', serif;
		}

	或者用内联式的 style ：

		<div style="font-family: 'Font Name', serif;">Your text</div>

> 注意，推荐在 `href` 中使用 https 的 url ，否则可能无法连接到 googleapis.com 。

在 [Google Fonts](https://www.google.com/fonts) 中可以看到可用的字体列表。
