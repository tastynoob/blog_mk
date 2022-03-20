---
title: longan-gd32v vscode环境配置
tag: [riscv,vscode]
categories: 其它
---
longan-gd32v vscode环境配置
<!--more-->


首先你要在[vscode官网](https://code.visualstudio.com/)下载vscode最新版

安装完成后，点击右侧的插件栏

搜索 __“EIDE”__ 插件

![1.png](https://i.loli.net/2021/06/17/FLKfYGlbNxQZCmu.png)

点击安装即可，第一次使用可能需要.net环境支持

另外还需要辅助插件 "c/c++",搜索安装即可

重启vscode

vscode侧边栏就会出现eide图标，点击

![2.png](https://i.loli.net/2021/06/17/TycDsdN2S96XqZh.png)


你需要指定编译器才能实现对项目的编译    
如果你没有选择任何工具链，那么“设置工具链路径”的图标是一个红色叉叉  
点击设置工具链路径  
它会要求你选择如下编译器    
根据自己的实际情况来选择编译器路径  
这里我们选择设置riscv gcc路径

![4.png](https://i.loli.net/2021/06/17/eUzTLBc1i4ubJpI.png)

gd32 gcc工具链可以在[此地址](https://cloud.github0null.io/s/R4SY?path=%2F%E7%BC%96%E8%AF%91%E5%B7%A5%E5%85%B7)下载

下载xpack-riscv-none-embed-gcc工具链

解压

在eide插件里选择riscv工具链路径

路径即riscv gcc安装根目录

---------

接下来我们来新建项目

EIDE插件已经为你提前准备好了longan gd32v的开发模板

在EIDE插件中选择新建项目

选择内置项目模板，往下拉

![0](https://i.loli.net/2021/06/17/hMydrpcL7NDEg85.png)

选择新建gd32vf项目模板

如果配置无误，即可看到1如下界面


![S145M_55__LYV~_UWOM6NCE.png](https://i.loli.net/2021/06/17/64nSYRWxHuLEVta.png)

按下F7进行编译

点击侧边栏EIDE插件，就可以看到当前项目的信息

![_`RU_VE_4BN7_6PB0__XSJK.png](https://i.loli.net/2021/06/17/LYx9hUIPpbfjslg.png)

你可对该项目的头文件包含路径，源文件包含路径进行更改

接下来就可以开始愉快的开发了