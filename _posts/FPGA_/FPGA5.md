---
title: FPGA学习-5：仿真
tag: verilog
categories: fpga
---


仿真
<!--more-->


在实际FPGA开发过程中

我们不可能直接将代码烧录进板子里进行测试

一是我们无法观察到其内部逻辑变换

而是万一代码中的逻辑有问题

直接进行甚至会导致芯片损坏

毕竟FPGA无法像单片机那样可以利用串口打印等调试方法

FPGA调试只能靠仿真

依然是拿之前我们写的点灯代码做测试

首先，仿真需要安装modelsim仿真工具  

该软件的安装这里不做赘述

软件、环境变量都配置好后开始

----------

首先FPGA仿真需要一个测试模块

我们先添加一个测试模块 __test.v__

在进行仿真时，仿真软件就会模拟运行test.v这个模块

点灯程序中，我们有3个输出和1个输入

因此在模拟时我们需要给目标模块提供一个模拟的输入

我们使用延时功能来产生一个固定频率的输入


![1.png](https://i.loli.net/2021/05/23/FGt6rCfJQloSBpk.png)

-------
然后就是启动仿真


![2.png](https://i.loli.net/2021/05/23/mgvWcoSh5lnEODH.png)

选择如图菜单

再点击 __"Behavioral Simulatin"__

即行为级仿真

![3.png](https://i.loli.net/2021/05/23/FLbDaeKuZhJ3mwr.png)

选择测试代码

它会生成modelsim的仿真命令行

![4.png](https://i.loli.net/2021/05/23/WyTbw6m7XjdKCLt.png)

打开命令行，依次输入以上前3条指令即可启动modelsim

![5.png](https://i.loli.net/2021/05/23/FwxB2t6Q7q1TjAh.png)

在modelsim最下面的命令行一依次输入

add wave *

run 1ms

然后鼠标指向仿真窗口，按住ctrl+滚动鼠标滚轮

就能看见仿真的波形

![6.png](https://i.loli.net/2021/05/23/QhTlmc2CJXfj7D3.png)


可以右键某个信号，比如out信号

选择properties

将radix设置为“unsigned”即可选择信号的数据类型

效果如下

![7.png](https://i.loli.net/2021/05/23/OftkLJpxa95ymrb.png)

这样二进制数据就能以无符号整数显示