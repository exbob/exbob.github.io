---
title: 在博客添加返回顶部按钮
date: 2014-07-09T08:00:00+08:00
draft: false
toc:
comments: true
---


>添加后的效果如右下角。

## 1. 导入 jQuery

在 [http://jquery.com/download/](http://jquery.com/download/) 中下载压缩过的 jQuery ：jquery-1.11.1.min.js ,放入博客模板的相应目录下，比如 include/ 。

在 base.html 的 <head> 中添加：

    <script type="text/javascript" src="/template/include/jquery-1.11.1.min.js"></script>

## 2. 添加 div

在 base.html 中添加:

    <div id="back-top">
        <a href="#top" title="回到顶部"></a>
    </div>
    
## 3. 添加js代码

使用 js 响应窗口滚动事件和按钮的点击事件，其中 100 表示向下滚动 100 个像素时出现按md钮，800 表示使用 800ms 的时间滚动到顶部。

    <script type="text/javascript">
        $("#back-top").hide();
        $(document).ready(function () {
            $(window).scroll(function () {
                if ($(this).scrollTop() > 100) {
                    $('#back-top').fadeIn();
                } else {
                    $('#back-top').fadeOut();
                }
            });
            $('#back-top a').click(function () {
                $('body,html').animate({
                    scrollTop: 0
                }, 800);
                return false;
            });
        });
    </script>
    
可以将这段代码添加到 `<div id="back-top">` 段的后面。

## 4. 添加样式

此处使用一个背景透明的向上箭头图片 bg_up_arrow.png，将其放入 include/ 目录下。通过 `backgroud-color` 指定正常时和鼠标放上时的背景颜色，`border-radius` 指定圆角的半径。

    #back-top {
      position: fixed;
      bottom: 30px;
      right: 80px;
    }
    #back-top a {
      width: 54px;
      height: 54px;
      display: block;
      background: #ddd url(/template/include/bg_up_arrow.png) no-repeat center center;
      background-color: #e9e9e9;
      -webkit-border-radius: 7px;
      -moz-border-radius: 7px;
      border-radius: 7px;
      -webkit-transition: 1s;
      -moz-transition: 1s;
      transition: 1s;
    }
    #back-top a:hover {
      background-color: #c8c8c8;
    }
