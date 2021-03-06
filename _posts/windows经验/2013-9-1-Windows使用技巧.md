---
layout: post
title: Windows使用技巧
categories: 
 - Windows
author: 杨比轩
---

最近有点闲时间，所以和大家分享分享我个人觉得windows下的一些比较实用又很装逼的技巧，可能都没什么难度，但是个人感觉这些都是可以提升效率的东西。

我个人确实不怎么喜欢把大量的应用快捷方式撇在桌面上，这样桌面看起来很乱，而且效率确实不怎么高，每次找应用其实都要找半天。而且我桌面上有很多临时的文件和文件夹，这些东西和快捷方式混在一起确实不怎么看好。为此，我也算是一直在思考和解决这个问题，所以这一篇分享基本都是如何避免将大量的快捷方式扔在桌面上，同时又可以快速的启动应用程序。
#一、巧用path快速启动应用
不管什么技巧，能用windows自带的功能，都尽量的不使用插件和第三方的软件，这也算是一个大前提吧。其实，从win8开始，系统自带的搜索已经足够强大。直接Win + S，然后输入程序名回车基本就OK了，我之前也都是这么干的。但是后来发现，这么搞的话貌似每次输入的东西太多了。像QQ这种软件还好，要是阿里旺旺等等要较长中文名字的程序，那就比较慢了，所以左思右想觉得利用环境变量path来快速启动貌似可以完美的避免上述问题。

具体做法：
- 首先，新建一个文件夹，将常用的应用程序的快捷方式或者自己写的一些脚本扔进去，然后将其名字都改为2~3字母这种形式。
- 复制刚才新建的文件夹的路径，将其添加到系统的环境变量（我的电脑右键-属性-高级系统设置-高级选项卡-环境变量-path）path下，如图：

![环境变量设置](http://upload-images.jianshu.io/upload_images/1156415-b4364b4e58e33598.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

要是出问题，可以百度下如何添加环境变量path即可。
- 接下来设置工作基本完成了，之后需要添加新的应用程序进去的时候，只需要将其快捷方式拷进去即可。平时使用的时候，只需要Win + R ，然后输入两三个简单的字母即可启动对应的应用程序。
> 比如我将Eclipse的快捷方式改为ec，然后撇进设置好环境变量的文件夹。使用的时候，Win + R，然后输入ec后回车即可启动Eclipse。

![运行界面](http://upload-images.jianshu.io/upload_images/1156415-491b9671f8981f67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

未完待续。。。
