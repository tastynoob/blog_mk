---
title: FPGA学习-2：开发板及Verilog介绍
tag: verilog
categories: fpga
---

开发板及verilog介绍
<!--more-->


我们将使用sipeed公司设计的tang permier开发板进行开发学习


![tang.jpg](https://i.loli.net/2021/05/16/2AjUkY8CclodXbr.jpg)


这款开发板使用了国产EG4S20芯片

拥有2万多个逻辑门单元

价格仅100多，极具性价比

可以在上面跑riscv开源架构cpu核心

完全够用

IDE我们使用官方的安路TD软件

详细安装过程可去[sipeed文档](https://tang.sipeed.com/en/)上查看

-----

与开发程序类似

开发FPGA同样有编程语言

比如verilog，VHDL

这里使用verilog作为核心开发语言

如果你对C语言有一定的了解，有助于Verilog的快速上手。

下面是一个简单的计数器


```verilog
module top(
    input clk,
    output reg[15:0] out_cnt
);
initial begin
    out_cnt = 16'b0;
end
always@(clk) begin
    if (out_cnt == 16'd250)begin
        out_cnt =0;
    end
    else begin
        out_cnt = out_cnt + 16'b1;
    end
end
endmodule
```


verilog语言主要以模块为单位来描述一个电路结构   


描述电路的方式有行为级描述，数据流描述，结构化描述

越往后难度越高，更接近于原始电路设计，但逻辑控制越简单

verilog将复杂的电路功能抽象为一个个模块(module)

通过组合个各个模块来完成不同的电路

并且verilog采用自顶向下的方式进行程序设计


![0](https://i.loli.net/2021/05/16/HT79FAGSVdMCktQ.png)


一步步完成各个模块再进行组合即完成一个基本的FPGA项目
