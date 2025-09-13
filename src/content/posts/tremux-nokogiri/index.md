---
title: 'Termux安装nokogiri'
description: '在手机上termux架设ruby环境安装nokogiri插件'
pubDate: 2023-07-03
cover: Termux-featured.jpg
tags:
 - termux
 - nokogiri
---

在手机上termux架设ruby环境，遇上安装xml nokogiri插件出错，最后发现依赖库有问题，网上找了很多解决方案都不成功，最后经过一番斗争终于解决

``` shell
pkg install libxml2 libxslt libiconv
gem install nokogiri -- --use-system-libraries
```
