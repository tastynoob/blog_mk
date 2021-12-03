---
title: FPGA学习-3：环境搭建
tag: verilog
categories: fpga
---

环境配置

<!--more-->

* __环境配置：__

安路FPGA的环境比较好搭建    

直接去[sipeed下载站](https://dl.sipeed.com/shareURL/TANG/Premier/IDE)中即可下载

如果出现没有license的情况

参加[这篇帖子](https://bbs.sipeed.com/thread/506)

* __新建工程：__

![](https://i.loli.net/2021/05/17/LCiHQ9z8nlXUWae.png)

这是安路TD开发软件的界面

新建项目点击左上角菜单的 __“project”__

再点击 __"New Project"__

![2.png](https://i.loli.net/2021/05/17/teZAofU9Vs6Gk3m.png)

选择好芯片后，就可点击“OK”建立工程了

接下来右键Hierarchy，选择“新建文件 new source”

![3.png](https://i.loli.net/2021/05/17/ImO48k6eHpdi2ho.png)

![4.png](https://i.loli.net/2021/05/17/DYZTIWktd4BKhlE.png)

文件类型选择verilog

文件名称可以随意

文件保存地址默认在项目文件夹下

同时要勾选 __"add to project"__

这样才能添加到工程参与编译

![5.png](https://i.loli.net/2021/05/17/jxFgQNZqoTzspiL.png)

正确操作后新建的文件应该就会被添加到项目

并且成为 __"top module"__   

__这个非常重要__,verilog采用自顶向下的设计模式，每个项目都有一个 __“top module”__   

verilog编译时是以“top module”开始的 

比如

![6.png](https://i.loli.net/2021/05/17/tre3bhoMZgkFHB1.png)

当你再添加一个文件时    

就会发现PLL文件和start文件的图标不同    

此时，start为“top module”顶层模块   

而PLL作为下级模块

当然，可以右键一个模块选择 __“set as top”__

就可以把选中的模块作为顶层模块，而原来的顶层模块就会变为下级模块    

一个项目当中 __有且仅有__ 一个 "top module"

安路TD软件的环境配置就此结束