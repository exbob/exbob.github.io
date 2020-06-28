---
title: 用 google-code-prettify 实现代码语法高亮
date: 2014-07-11T08:00:00+08:00
draft: false
toc:
comments: true
---


> google-code-prettify 可以通过 javascript 和 css 文件为 html 页面实现代码段的语法高亮。

## 1. 下载

在 [https://code.google.com/p/google-code-prettify/](https://code.google.com/p/google-code-prettify/) 下载 prettify-small-4-Mar-2013.tar.bz2 ，解压后的文件夹放在博客模板的 include 目录下，改名为 prettify 。

## 2. 导入

在模板的 base.html 文件的 `<head>` 段添加 ：

    <link href="/template/include/prettify/prettify.css" type="text/css" rel="stylesheet" />

js 文件添加到 `<body>` 段的尾部，这样可以提升加载速度 ：

    <script type="text/javascript" src="/template/include/prettify/prettify.js"></script>
    
## 3. 实现代码高亮

在 base.html 文件的 `<body>` 段尾部添加如下代码，它会在页面的 `<pre>` 标签添加 `class="prettyprint linenums prettyprinted" style="overflow: auto;"` ，用于识别需要高亮的代码块 ：

    <script type="text/javascript">
        $(document).ready(function() {
            $('pre').addClass('prettyprint linenums').attr('style', 'overflow:auto');
            prettyPrint();
        });
    </script>

> 这段 js 代码用到了 jQuery ，需要提前引入。

如果对代码高亮的主题不满意，可以修改 prettify.css 文件。
