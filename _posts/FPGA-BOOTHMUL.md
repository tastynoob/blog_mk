---
title: BOOTH乘法器的verilog实现
tag: FPGA
categories: 技术
---

BOOTH乘法器的verilog实现
<!--more-->

booth乘法器是一种有符号整数乘法器
下面是它的多周期实现
至于原理就懒得讲了


```verilog
//booth多周期乘法器
module BOOTH_MUL (
    input wire i_clk,
    input wire i_rstn,
    input wire[31:0] A,
    input wire[31:0] B,
    input i_en,
    output wire[63:0] Y,
    output wire o_finish
);
    reg[64:0] p;
    reg finish;
    reg[15:0] cnt;
    reg[1:0] tag;
    initial begin
        p<=0;
        finish<=0;
        cnt<=0;
        tag<=0;
    end
    wire[31:0] _A = 0-A;//被乘数B的负数
    wire b0 = p[0];
    wire b1 = p[1];

    //p的高32位
    wire[31:0] ph = b0^b1 ? (p[64:33] + (b1 ? _A : A)) : p[64:33];

    always @(posedge i_clk or negedge i_rstn) begin
        if(i_rstn == `rst)begin
        end
        else begin
           case(tag)
                0:begin
                    if(i_en)begin
                        p <= {32'b0,B,1'b0};
                        finish<=0;
                        cnt<=0;
                        tag<=1;
                    end
                end
                1:begin
                    cnt <= cnt + 1;
                    p <= {ph[31],ph,p[32:1]};
                    if(cnt == 31)begin//这里是并行判断
                        tag <=2;
                        finish<=1;
                    end
                end
                2:begin
                    tag<=0;
                    finish<=0;
                end
           endcase 
        end
    end

    assign o_finish = finish;
    assign Y = p[64:1];
endmodule
```