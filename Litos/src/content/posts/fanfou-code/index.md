---
title: '饭否Flash插件'
pubDate: 2008-09-20
description: '嘿嘿，做了个饭否Flash插件，放在博客上或者qq空间，美滋滋.'
tags:
  - Flash
  - Code
  - QQ空间
cover: "as.jpg"
---



嘿嘿，做了个饭否Flash插件，放在博客上或者qq空间，美滋滋

饭否是什么？

饭否是一个迷你博客。你可以通过手机、网页、MSN/GTalk/QQ随时随地发消息，时时刻刻看朋友。

注册饭否：[http://fanfou.com](http://fanfou.com)

## 饭否Flash插件

功能:通过api的方式把饭否即时信息显示在博客或者qq空间页面

## 使用说明：

页面中嵌入以下HTML代码：

```html
<embed flashvars="id=饭否ID" bgcolor="#001342" allowscriptaccess="sameDomain" wmode="Transparent" height="130" loop="false" menu="false" name="FanShow" pluginspage="http://www.macromedia.com/go/getflashplayer" quality="high" src="https://iiloveyou.org/swf/fanshow.swf" mce_src="https://iiloveyou.org/swf/fanshow.swf" type="application/x-shockwave-flash" width="335"/>
```

## 定制的部分：

flashvars：“饭否ID”
wmode：”Transparent”，# 背景为透明色。
bgcolor：”#001342” #背景颜色，如果需要显示背景颜色，而wmode值为空。


直接使用swf（如qq空间）调用的Flash地址代码：
```
https://iiloveyou.org/swf/fanshow.swf?id=饭否ID
```