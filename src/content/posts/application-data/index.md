---
title: "Application Data转移"
description: "修改Application Data目录到D盘，重装系统恢复软件数据"
pubDate: 2008-10-26
cover: windows.jpg
postType: coverSplit
tags:
  - Win
  - 电脑
  - 数据
---

其实使用过Chrome Firefox Safari等浏览器都觉得比IE好用（IE早已经有了方法），但我的系统是安装了还原软件的，每次重启系统后这些浏览器的设置以及用户数据也会随着还原而丢失，这令我非常苦恼，所以也一直没有常常使用这些浏览器，通常是用来测试网页的兼容性，今天实在是无法忍受这些痛苦了，花了点时间研究了一下修改了Application Data文件夹路径，方法如下：

打开你所使用的帐户文件夹（路径C:Documents and Settings\你的用户名），通常会是使用超级用户名字一般为Administrator将其在资源管理器中的“应用程序信息Application Data以及C:Documents and SettingsLocal Settings下的Application Data（此两个文件夹要取消隐藏选项才能看到），复制到其他分区（即非还原分区）。

```
路径：C:Documents and Settings用户名\Local Settings\Application Data

路径：C:Documents and Settings用户名\Application Data
```

复制到其他分区例如：

```
路径：D:用户名\Local Settings\Application Data

路径：D:用户名\Application Data
```

注意：可能部分文件因为使用中无法复制，请忽略或者直接复制里面子文件夹.例如：FF的Mozilla  Safari的Apple Computer等

接着在“开始”菜单中选择“运行”，输入“regedit”，然后点击“确定”按钮打来注册表。在打开的“注册表编辑器”窗口左侧，寻找并选择
```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
```
双击右侧的“AppData”，将其数据数值改为“你所复制的其他分区位置”
例如：
```
D:Administrator\Application Data
```
点击“确定”按钮。
双击右侧的“Local AppData”，将其数据数值改为“你所复制的其他分区位置”
例如：
```
D:Administrator\Local SettingsApplication Data
```

退出注册表窗口，重新启动电脑就ok了

当然如果你不想保留系统盘中的Application Data 你也可以找到相应的目录将其删除，建议还是不要删除，保留备用吧

请修改前做好备份，以防不测，虽然本人已经拿自己的电脑测试过了
