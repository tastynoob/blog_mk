---
title: FPGA学习-4：初步入门点个灯
tag: verilog
categories: fpga
---

点灯
<!--more-->

环境配置好后就正式进入verilog的学习了

先从最简单的点灯开始

我会向你们介绍最基础的数据类型和语法规则

![1](https://i.loli.net/2021/05/18/HUFIkWMdonACr6h.png)


仍然是从我们熟悉的界面开始

转到顶层模块 start中

-----


首先，verilog都是以模块为单位进行编程

模块以 __“module” “endmodule”__ 作为界限

如图既是定义一个模块start,即顶层模块

当然，可以定义其它模块再设置为顶层模块

------

首先，一个电路模块肯定有输入输出    

还有寄存器，逻辑控制什么的

先附上点灯代码

```verilog
module start(
	input button,//按键输入
	output[2:0] led//3个led输出
 );
reg[2:0] dat=3'b111; //3个led的状态寄存器
assign led = dat;//将寄存器的3个值连接到led输出

//当检测到按键下降沿时触发
always@(negedge button) begin
    //下面与c语言类似
    //但也有很大的区别
	if(dat == 3'b000)begin
		dat = 3'b111;
	end
	else begin	
		dat = dat - 3'b1;	
	end
end
//结束
endmodule
```


这里看不懂没关系

后面再慢慢解释

先将这些代码写入顶层模块

![2](https://i.loli.net/2021/05/18/3QXlZqsNWxiJGav.png)

然后保存一遍， __注意：一定要保存！！__

既然有了代码，那么该如何点灯呢？

既不像51直接“led=1”

也不直接操作寄存器

__这里FPGA采用了一种管脚映射的方式__


打开TD软件的“Tools”菜单,点击“IO Constraint”

即可看到引脚设置界面


![3](https://i.loli.net/2021/05/18/rmpOIGRy7JjXDxS.png)

图正中的即芯片管脚图

而下面就会出现我们定义的几个输入输出通道

分别是:

button 对应K16引脚

led\[0~2]分别对应R3，J14，P13这三个引脚

从开发板的原理图中可以看到

![4](https://i.loli.net/2021/05/18/X8BDybFMKmJHpnE.png)


K16引脚对应“user”按钮

R3,J14,P13对应led_red,green,blue

利用管脚映射即可实现verilog模块对外部引脚的控制

设置好引脚后，即可关闭引脚编辑器

__关闭时，它会提示你进行保存，点击确定，并保存引脚设置文件__

此时，你的项目中就会多出一个 __引脚设置文件：“.adc”__

![5](https://i.loli.net/2021/05/18/MgRLCrn4E65idmT.png)


这样，点灯程序就完全配置完成了

接下来就是编译，烧录

![6](https://i.loli.net/2021/05/18/wNtkhzJ2y7p4W8E.png)

点击此按钮进行编译

如果提示：没有license

参加[这篇帖子](https://bbs.sipeed.com/thread/506)

----
接下来就是烧录

使用micro-usb线插上tang permier

烧录要更新驱动


![7](https://i.loli.net/2021/05/18/sD9leIkfBW6VAqQ.png)


如果没有该设备

那么可能是这2个中的任意一个

![no_driver_win10.png](https://i.loli.net/2021/05/18/6TZRx4SKYPrCXq8.png)

![no_driver.png](https://i.loli.net/2021/05/18/cSeLGFpCkjTEOXx.png)


选择最可能是tang permier的设备

右键-更新驱动程序-浏览我的电脑以查找驱动程序

![7](https://i.loli.net/2021/05/18/1wVe7K2sJRlfmDN.png)

选择安路TD软件安装目录下的driver里选择正确的驱动

我的电脑是win10 64位，因此选择win8_10_64驱动

点击下一步进行安装

---

驱动更新完成后

再打开安路TD

![8](https://i.loli.net/2021/05/18/DuFmqKhApT2dVQc.png)

点击此按钮进入烧录界面

![9](https://i.loli.net/2021/05/18/XKJ6qxm7p3VTnEY.png)

根据图示进行烧录即可完成

---------

烧录完成后

按下板子上的user按钮，就会发现led出现变换闪烁

主要还是因为按键抖动的原因导致时序错误

这个可以不用担心芯片是否会损坏