---
title: FPGA进阶：流水线结构
tag: verilog
categories: fpga
cover: /images/lsp/4.jpg
---

无
<!--more-->

了解了状态机结构，接下来就来学习更加高级的流水线结构

状态机结构的好处是简单方便，但是综合出来的电路时序不高

也就是很难高频工作，因为在一个时钟里要进行多次判断匹配

虽然代码看上去很简单，但是综合出来的电路十分冗杂

通常用于时序要求不高简单的电路

---------------

流水线则与之相反

流水线将一个工作拆分成多个小任务

每个小任务执行起来十分简单快速

每个小任务就是一个小模块

这样有利于时序，但是会增加电路面积和复杂度

最常见的就属cpu电路，频率动辄上Ghz

下面是一个经典的5级流水线cpu运行时序

![1](https://i.loli.net/2021/07/07/U3ojqnNagyEZd8V.png)

----------------


接下来我们来使用流水线结构实现一个简单的整数乘法器

![2](https://i.loli.net/2021/07/07/txLmBuVShNMPQgD.png)

代码如下

```verilog
module mul_cell
    #(parameter N=4,
      parameter M=4
    )(
      input clk,        //时钟信号
      input rstn,       //复位信号
      input en,         //使能信号
      input[M+N-1:0] mult1,      //被乘数
      input[M-1:0] mult2,      //乘数
      input[M+N-1:0] mult1_acci, //上次累加结果
      output reg[M+N-1:0] mult1_o,     //被乘数移位后保存值
      output reg[M-1:0] mult2_shift, //乘数移位后保存值
      output reg[N+M-1:0] mult1_acco,  //当前累加结果
      output reg rdy          //预备信号
    );
    always @(posedge clk or negedge rstn) begin
        if (!rstn) begin
            rdy            <= 'b0 ;
            mult1_o        <= 'b0 ;
            mult1_acco     <= 'b0 ;
            mult2_shift    <= 'b0 ;
        end
        else if (en) begin
            rdy            <= 1'b1 ;
            mult2_shift    <= mult2 >> 1 ;
            mult1_o        <= mult1 << 1 ;
            if (mult2[0]) begin
                //乘数对应位为1则累加
                mult1_acco  <= mult1_acci + mult1 ;  
            end
            else begin
                mult1_acco  <= mult1_acci ; //乘数对应位为1则保持
            end
        end
        else begin
            rdy            <= 'b0 ;
            mult1_o        <= 'b0 ;
            mult1_acco     <= 'b0 ;
            mult2_shift    <= 'b0 ;
        end
    end
endmodule

module mul_main
    #(parameter N=4,//乘数1位数
      parameter M=4//乘数2位数
    )(
      input clk,
      input rstn,
      input data_rdy ,
      input[N-1:0] mult1,
      input[M-1:0] mult2,
      output res_rdy,
      output[N+M-1:0] res //输出
    );

    wire [N+M-1:0]       mult1_t [M-1:0] ;
    wire [M-1:0]         mult2_t [M-1:0] ;
    wire [N+M-1:0]       mult1_acc_t [M-1:0] ;
    wire [M-1:0]         rdy_t ;

    //第一次例化相当于初始化，不能用 generate 语句
    mul_cell #(
        .N(N), 
        .M(M)
    )
    u_mult_step0(
        .clk              (clk),
        .rstn             (rstn),
        .en               (data_rdy),
        .mult1            ({{(M){1'b0}}, mult1}),
        .mult2            (mult2),
        .mult1_acci       ({(N+M){1'b0}}),
        //output
        .mult1_acco       (mult1_acc_t[0]),
        .mult2_shift      (mult2_t[0]),
        .mult1_o          (mult1_t[0]),
        .rdy              (rdy_t[0]) 
    );

    //多次模块例化，用 generate 语句
    genvar i;
    generate
        for(i=1; i<=M-1; i=i+1) begin
            mul_cell #(
                .N(N),
                .M(M)
            )
            u_mult_step(
                .clk(clk),
                .rstn(rstn),
                .en(rdy_t[i-1]),
                .mult1(mult1_t[i-1]),
                .mult2(mult2_t[i-1]),
                //上一次累加结果作为下一次累加输入
                .mult1_acci(mult1_acc_t[i-1]),
                //output
                .mult1_acco(mult1_acc_t[i]),                                      
                .mult1_o(mult1_t[i]),  //被乘数移位状态传递
                .mult2_shift(mult2_t[i]),  //乘数移位状态传递
                .rdy(rdy_t[i]) 
            );
        end
    endgenerate

    assign res_rdy = rdy_t[M-1];
    assign res = mult1_acc_t[M-1];

endmodule

`timescale 1ps/1ps
module test ();
    reg clk=0;
	reg[7:0] A=0;
	reg[7:0] B=0;
	

    always begin
        #1 clk = ~clk;
    end
	always @(posedge clk) begin
		A = A + 1;
		B = B + 2;
	end
	wire[15:0] C;
	wire rdy;
    mul_main #(
        .N(8),
		.M(8)
    )
    mul(
        .clk(clk),
        .rstn(1'b1),
        .data_rdy(1'b1),
        .mult1(A),
        .mult2(B),
        .res_rdy(rdy),
        .res(C)
    );

endmodule

```

仿真如下

![3](https://i.loli.net/2021/07/07/ENv73HbWtPQc9MB.png)


可以看到，在一开始隔了几个时钟周期后

乘法结果开始以流水形式输出，效率非常高

流水线的优点不言而喻