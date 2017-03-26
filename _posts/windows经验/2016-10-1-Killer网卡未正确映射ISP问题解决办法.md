---
layout: post
title: Killer网卡未正确映射ISP问题解决办法
categories: 
 - Windows
author: 杨比轩
---

微星和外星人的部分电脑都用的是杀手网卡，安装完成后每次电脑启动都会出现下图错误提示：


![错误提示](http://upload-images.jianshu.io/upload_images/1156415-17b5abebe65d9c02.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实这个问题应该是无伤大雅的毛病，但是每次开机都提示就是让人不爽，总感觉系统哪里有问题，所以就想办法解决了下这个问题。最后发现是系统的一个关键性服务被第三方软件禁用了，设置为自启动就好了。

解决办法：
- 计算机管理-服务
- 找名为**Link-Layer Topology Discovery Mapper**服务，设置为自动启动就好了。
