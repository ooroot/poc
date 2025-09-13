---
title: "用JS+CSS的方法左右对齐的DIV列表"
pubDate: 2009-10-24
description: '当调用数据库输出一个<li>列表横向排列时，而这个列表我想左右两边靠边（左右对齐父div），这样就需要对<li>做不同的class名以定义不同的样式。当然实现这样的效果有很多.'
cover: "js-li-css.jpg"
categories:
    - Code
tags:
  - Code
  - CSS
  - JS
---


有些日子没有贴过代码了，都有些忘记了，写这类文章还是很懒的，测试写备注贴代码做演示等等都是让人抓狂的。

前些日子在制作主题遇到一个问题，当调用数据库输出一个<li>列表横向排列时，而这个列表我想左右两边靠边（左右对齐父div），这样就需要对<li>做不同的class名以定义不同的样式。当然实现这样的效果有很多解决的方法，但我用了一段JS规则动态修改class（类名）。

优点：省事/快捷/动态

缺点：js是最后面执行的，所以在页面其他内容未加载完成时，不能处理js，所以会造成短暂的样式失效，以及有部分用户的浏览器是禁止js的（反正我没见过这么变态的），这样就造成永远样式失效。

废话不多说，开始吧，注：class=box id=list   (i%4>0) 等可自定义

html部分：调用数据库输出的列表

```html
<div class=box id=list><!-- 调用数据库输出的列表-->
 <ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
  <li>4</li>
  <li>5</li>
  <li>6</li>
  <li>7</li>
  <li>8</li>
 </ul>
</div>
```
js部分：目的修改box的li className

```js
<script type="text/javascript">
<!--
var Ptr=document.getElementById("list").getElementsByTagName("li");//指定的id 属性值得到对象
function $() {
      for (i=1;i<Ptr.length+1;i++) {
      Ptr[i-1].className = (i%4>0)?"left":"right";//修改className (i%4>0)为节点
      }
}
window.onload=$;
//-->
</script>
```
html经过js动态修改后结果：

```html
<div class=box id=list>
 <ul>
  <li class="left">1</li><!-- 其他li className 均为left-->
  <li class="left">2</li>
  <li class="left">3</li>
  <li class="right">4</li><!-- js规则每逢第四处添加li className为right-->
  <li class="left">5</li>
  <li class="left">6</li>
  <li class="left">7</li>
  <li class="right">8</li>
 </ul>
</div>
```
这样我们就获得到li的class=”left”和class=”right”两个不同的类名，这样我们就能为两组类名定义样式

css部分：给列表附上样式

```css
.box {background: #dedede;margin:0 auto;overflow:hidden;}
ul{ padding:0; margin:0;}
.box ul li {list-style:none;height:100px;width:100px;}
.box ul li.left {background-color:#666; margin:0 5px 5px 0; float:left;}/* 左样式 */
.box ul li.right {background-color:#999;margin:0 0 5px 0; float:right;}/* 右样式 */
```

[演示：JS+CSS+DIV](./js-li-css.html)