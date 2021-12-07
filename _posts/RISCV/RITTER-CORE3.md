---
title: FPGA-RISCV内核入门3
tag: riscv
categories: FPGA
---

接下来我们就开始正式的设计RISCV cpu了

<!--more-->

之前说了

我们所设计的rv32i内核使用五级流水线

所以我们来了解下cpu流水线的设计方法


![_NMXW@@APF_ZHALC_Y7__UL.png](https://i.loli.net/2021/09/12/56OxuL1aBydTWA3.png)

这是一个最简单的流水线模型

中间的可以是组合电路或者时序电路

第一个时序电路在上升沿时读取上一层电路的输出

经过中间组合/时序电路的计算后输出到第二个时序电路

在第二个上升沿来时，第二个时序电路读取中间输出，第一个时序电路读取上层输出

如此往复。

假如中间的运算要花费多个周期，还需要将流水线暂停

多个这样的单元组成就构成了基本的流水线

对于cpu内核也是一样

即将一条指令拆分成多个步骤，然后交给流水线执行

可以确保每条指令的执行时间为1个周期，同时大大提高了时序

对于五级流水线内核：取指，译码，执行，访存，写回

取指段是时序逻辑

译码段是组合逻辑

执行段可以是组合或时序逻辑

访存是时序逻辑

写回是组合逻辑

要将每一层连接起来就要靠上图基本流水线的1级时序和2级时序电路，也可以称之为连接段

下面是取指段到译码段的连接段


```verilog
//取指与译码模块之间的隔离模块
module IF_ID (
    input wire i_clk,
    input wire i_rst_n,
    
    //流水线暂停
    input wire i_pipe_stop,
    //流水线冲刷
    input wire i_pipe_flush,

    

    //上级输入的指令地址
    input wire[31:0] i_iaddr,
    //ibus输出的指令数据,
    input wire[31:0] i_idata,


    //传递给下级的指令地址
    output reg[31:0] o_iaddr,
    //输出的指令数据
    output reg[31:0] o_idata

);
    wire en =  (~i_pipe_stop | i_pipe_flush);

    initial begin
        o_iaddr<=0;
        o_idata<=0;
    end
    always @(posedge i_clk or negedge i_rst_n) begin
        if(i_rst_n == `rst)begin
            o_iaddr <= 32'b0;
            o_idata <= `inst_nop;
        end
        else if(en == `en)begin

            o_iaddr <= (i_pipe_flush == `en) ? 32'h0 : i_iaddr;
            o_idata <= (i_pipe_flush == `en) ? `inst_nop : i_idata;
           
        end
    end


endmodule
```


下一节我们将会来了解下简单的取指过程

