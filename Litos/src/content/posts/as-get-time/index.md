---
title: 'W友给的一段修改时区的代码'
pubDate: 2008-09-22
description: 'Flash通过as获取当前时间，从而计算过去了多长时间.'
tags:
  - Code
  - AS
cover: "as.jpg"
---

由于获取到的时间是标准格式(Mon, 22 Sep 2008 01:13:02 +0000),所以可以用new Date(时间字符串)来得到标准时间对象. 在MsgListItem类中有处理函数

```js
public function perseTime(time:String):String {
var resultTime:String="";
date=new Date(time);
nowDate=new Date();
var c=nowDate.getTime()-date.getTime();
if (c<millisecondsPerDay) {//24小时内
if (c<millisecondsPerMinute) {//1分钟内
resultTime=Math.round(c/millisecondsPerSecond)+"秒前";
} else if (c<millisecondsPerHour) {//1小时内
resultTime=Math.round(c/millisecondsPerMinute)+"分钟前";
} else {//1小时以上
resultTime="约"+Math.round(c/millisecondsPerHour)+"小时前";
}
} else {
resultTime=date.fullYear+"年"+(date.month+1)+"月"+(date.date)+"日 "+date.toTimeString();
}
return resultTime;
```

又找到這樣一段

```js
function getDateFromString(str)
{
var _loc3 = {Jan: 0, Feb: 1, Mar: 2, Apr: 3, May: 4, Jun: 5, Jul: 6, Aug: 7, Sep: 8, Oct: 9, Nov: 10, Dec: 11};
var _loc1 = str.split(" ");
var _loc2 = _loc1[3].split(":");
return (new Date(Date.UTC(_loc1[5], _loc3[_loc1[1]], _loc1[2], _loc2[0], _loc2[1], _loc2[2])));
} // End of the function
function relative_time(time_value)
{
var _loc3 = getDateFromString(time_value);
var _loc2 = new Date();
var _loc1 = (_loc2.getTime() - _loc3) / 1000;
switch (true)
{
case _loc1 < 60:
{
return ("约 1 分钟前");
break;
}
case _loc1 < 120:
{
return ("约 1 分钟前");
break;
}
case _loc1 < 2700:
{
return (int(_loc1 / 60) + " 分钟前");
break;
}
case _loc1 < 5400:
{
return ("约 1 小时前");
break;
}
case _loc1 < 86400:
{
return ("约 " + int(_loc1 / 3600) + " 小时前");
break;
}
case _loc1 < 172800:
{
return ("1 天前");
break;
}
default:
{
return (int(_loc1 / 86400) + " 天前");
break;
}
} // End of switch
} // End of the function
```