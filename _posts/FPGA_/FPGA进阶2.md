---
title: FPGA进阶：状态机结构
tag: verilog
categories: fpga
---

无
<!--more-->


在FPGA编程中

由于FPGA本身不能像c语言那样顺序执行

也没有各种方便函数可以调用

在设计FPGA顺序执行时就会有很大的困难

因此这里介绍FPGA的状态机机制

状态机结构几乎可以用于所有的数字逻辑当中

大大减小了开发难度

下面是一个基本的顺序循环状态机

![0](https://i.loli.net/2021/06/17/iObLtFRZnYC4EGH.png)


可以用来模拟c语言结构中的 __“while(1)”__ 循环

verilog代码演示：

```verilog
module test (
    input clk,
    output reg[3:0] line
);

    integer tag = 0;
    always @(posedge clk) begin
        case(tag)
            0:begin
                line=4'b0001;
                tag = 1;
            end
            1:begin
                line=4'b0010;
                tag = 2;
            end
            2:begin
                line=4'b0100;
                tag = 3;
            end
            3:begin
                line=4'b1000;
                tag = 0;
            end
        endcase
    end
    
endmodule
```

其中tag用于状态指示器

clk时钟信号作为运行节拍

在这种基本状态机基础上可以衍生出其它状态机

这里利用状态机来设计一个石头剪刀布小游戏

状态机结构如下


![0](https://i.loli.net/2021/06/17/8IgS1v2u7kw6AlL.png)


verilog代码如下

```

module test (
    input clk,
    input[2:0] a,
    input[2:0] b,

    output reg a_light,
    output reg b_light
);

    integer tag = 0;
    integer G ;
    always @(posedge clk) begin
        case(tag)
            0:begin
                G=10;
                tag = 1;
                a_light=0;
                b_light=0;
            end
            1:begin//比较
                if(a>b)begin
                    tag = 2;
                end
                else if(a<b) begin
                    tag = 3;
                end
                else begin
                    tag = 4;
                end
            end
            2:begin//a赢
                G  = G + 1;
                tag = 5;
            end
            3:begin//b赢
                G = G - 1;
                tag = 6;
            end
            4:begin//平
                if(G>10)begin
                    G = G - 1;
                end
                else if(G<10) begin
                    G = G + 1;
                end
                tag = 9;
            end
            5:begin
                if(G >= 20)begin
                    tag = 7;
                end
                else begin
                    tag = 1;
                end
            end
            6:begin
                if(G <= 0)begin
                    tag = 8;
                end
                else begin
                    tag = 1;
                end
            end
            7:begin//亮a灯
                a_light=1;
                b_light=0;
                tag = 0;
            end
            8:begin//亮b灯
                a_light=0;
                b_light=1;
                tag = 0;
            end
            9:begin//亮ab灯
                a_light=1;
                b_light=1;
                tag = 1 ;
            end
        endcase
    end
    
endmodule

```

























