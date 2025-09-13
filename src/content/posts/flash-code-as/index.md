---
title: 'Flash即拷即用的加载动画代码'
description: 'Flash即拷即用的加载动画代码'
pubDate: 2008-09-22
cover: "as.jpg"
tags:
    - Code
    - AS
    - JS
---


复制下面的代码，放在第1帧即可
```js
// 设置loading的宽、高、颜色
var loading_width = 200;
var loading_height = 3;
var loading_color = 0x000099;
var rect1 = createRectangle(this, loading_width, loading_height, loading_color, true);
var rect2 = createRectangle(this, loading_width, loading_height, loading_color, false);
rect1._x = (Stage.width - loading_width) / 2;
rect1._y = (Stage.height - loading_height) / 2;
rect2._x = rect1._x;
rect2._y = rect1._y;
this.onEnterFrame = function ()
{
rect1._width = _root.getBytesLoaded() / _root.getBytesTotal() * loading_width;
if (_root.getBytesLoaded() == _root.getBytesTotal()) {
rect1.removeMovieClip();
rect2.removeMovieClip();
delete rect1;
delete rect2;
delete this.onEnterFrame;
play();
}
};
function createRectangle(scope, w, h, color, is_fill)
{
var l = scope.getNextHighestDepth();
var _mc = scope.createEmptyMovieClip("mc_" + l, l);
with (_mc) {
lineStyle(0,color,100);
if (is_fill) {
beginFill(color,100);
}
lineTo(0,h);
lineTo(w,h);
lineTo(w,0);
lineTo(0,0);
endFill();
}
return _mc;
}
stop();  
```
来源：http://www.flashxm.com/?p=39