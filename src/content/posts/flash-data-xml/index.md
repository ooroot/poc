---
title: '关于XML在FLASH交互应用中的理解'
description: '关于FLASH解析XML详解，说白了，也就是对节点的读取。'
pubDate: 2008-09-26
cover: "as.jpg"
tags:
  - Code
  - Flash
  - AS
---

## 一、书写标准的XML需要注意以下几点：

1. 一个标准的XML文档必须要在第一行以"<?xml>"开头，以及一个"<?/>"结尾，其中开头部分需申明XML的版本，如：version="1.0"，也应包含一个关于编码的声明：“encoding="utf-8"”；
2. XML要求所有的标签必须在文档结束之前被闭合，如果标签中包含内容，则必须写成“<标签名>内容</标签名>"的格式，如果内容为空，则应写在</标签名>的格式；即，后面必须有</XXX>这种格式的。
3. 每个XML都有有一个根元素，也叫根节点。
4. 节点有节点值和属性。
5. XML的节点值如果出现与XML标签关键字一样的字符，XML将会无法解析文档，比如，某个节点值如果是一个大于或者小于号，就会让解释器产生一个错误。

## 二、在FLASH中创建一个简单的实例：

```js
//创建一个新的XML类实例
var my_XML:XML=new XML();
//创建一个TextArea组件，用于显示此XML内容
var my_TextArea:mx.controls.TextArea;
//加载外部XML文件
my_XML.load("my_xml.xml");
//为load函数定义onLoad函数---此处是必须的。只有当加载成功后，才能控制XML。否则不能。
my_XML.onLoad=function(ok:Boolean){
if(ok){
my_TextArea.text+=this;
}else{
my_TextArea.text+="加载失败..."
}
}
//勿略空白：
my_XML.ignoreWhite=true;
```

## 三、关于FLASH解析XML详解，说白了，也就是对节点的读取。

下面，我将对XML文件的解析方面做详细的介绍。为了便于讲解，在此，我举一个简单的例子。
1. 新建一个记事本文件，在里面输入如下内容：

```xml
<衬衣 颜色="白色" 品牌="雅戈尔">1件
<裤子 颜色="深蓝" 品牌="王子裤">2件
<鞋子 颜色="黑色" 品牌="红蜻蜓">3双
<总花费金额>
<打的费>50元
<货物金>1000元
```

然后保存为"goods.xml"，注意，在编码处，要选择"Unicode"。

2. 下面，就对里面的各项值进行解读。
读取整个XML文件的方法：在刚才的XML的同级目录下新建一FLA文件，然后在场景中放置一TextArea组件。并赋实例名“my_TextArea”，然后在第一帧中添加如下代码：

```js
var my_XML:XML=new XML();
System.useCodepage=true;
var my_TextArea:mx.controls.TextArea;
my_XML.load("goods.xml")
my_XML.onLoad=function(ok:Boolean){
if(ok){
my_TextArea.text+=this;
}else{
my_TextArea.text+="加载失败"
}
} 
```

按CTRL+ENTER测试，即可以看效果。
首先，我们来看看如何对XML中的节点名称或者节点属性进行读取呢？比如，我们要读取如例子中的"购买物品"这几个字符，该如何操作呢？
我们使用如下语句：
this.firstChild.nodeName;
如果要读取"衬衣"：
this.firstChild.childNodes[0].nodeName;
如果要读取"1条"：
this.firstChild.childNodes[1].childNodes[0].nodeValue;
如果要读取裤子的品牌：
this.firstChild.childNodes[1].attributes.品牌；

将XML读取进LIST组件中：
 
```js
System.useCodepage = true;
var my_TextArea:mx.controls.TextArea;
var my_List:mx.controls.List;
var my_XML:XML = new XML();
my_XML.ignoreWhite = true;
my_XML.load("shopping.xml");
my_XML.onLoad = function(ok:Boolean) {
if (ok) {
var childnodes = this.firstChild.childNodes;
for (i=0; i<childnodes.length; i++) {
my_List.addItem({label:childnodes.nodeName});
}
} else {
my_TextArea.text += "加载失败";
}
};
```
第一个节点：firstChild:
用法：this.firstChild;
最后一个节点：lastChild
用法：this.firstChild;
兄弟节点：nextSibling:
用法：this.firstChild.childNodes[0].nextSibling;
另一个兄弟节点：previousSibling:
用法：this.firstChild.childNodes[0].previousSibling;
区别：nextSibling是朝后,而previousSibling是当前之前;
父类节点：parentNode:
用法：this.firstChild.parentNode;