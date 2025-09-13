---
title: '发布Z-blog主题-淡雅E'
pubDate: 2008-10-24
description: 'Z-blog主题-淡雅E，总体感觉很简洁，有种流线美，呵呵,欢迎大家试试，已经在各大浏览器中测试过了.'
tags:
  - zblog
  - 主题
  - 淡雅E
cover: "z-blog-e.jpg"
---

发布Z-blog主题-淡雅E，总体感觉很简洁，有种流线美，呵呵,欢迎大家试试，已经在各大浏览器中测试过了。

需要注意：后台设置（正文图片自动缩放宽度）宽度为442，其实是450，因为图片加边框去掉左右各4px


彻底解决IE6问题，支持目前几乎所有浏览器（IE6-7-8 FF3 Google chrome）
修正长标题问题，gravatar头像功能

## 2009-01-03更新
支持宽屏（发现自打金融风暴以来，我的博客近一半的访客都使用了宽屏 ）
去掉顶部的图片Logo，换成文本
侧栏列表加入ico
修复内文图片样式
标题换了黑体

##  2009-02-14更新
修正图片链接错误代码
添加锚点跳转缓冲

CSS中添加了导航条按钮焦点样式 》[haphic](http://www.esloy.com/blog/about/) 完善后的代码
请在后台导航条中加入下面的代码：

```js
<script type="text/javascript">
$("#divNavBar ul>li").each(function() {
if ($(this).find("a").attr("href").toLowerCase() !== str00.toLowerCase()){
if (document.URL.toLowerCase().indexOf($(this).find("a").attr("href").toLowerCase()) !== -1){
$(this).attr("id","current")
}
}else{
if ($(this).find("a").attr("href").toLowerCase() == document.URL.toLowerCase()){
$(this).attr("id","current")
}
}
});
</script>
```

下载：[淡雅E](https://1drv.ms/u/s!Aomk5GCWnNqagRjBRSaQV-2U5LdS)
