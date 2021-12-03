---
title: 无符号除法器的verilog实现
tag: FPGA
categories: 技术
---

无符号符号器的verilog实现
<!--more-->
下面是它的多周期实现


```verilog
module DIV (
    input wire i_clk,
    input wire i_rstn,
    input wire[31:0] A,
    input wire[31:0] B,
    input i_en,
    output wire[31:0] Y,
    output wire[31:0] REM,
    output wire o_finish
);
    reg finish;
    reg[2:0] flag;
    reg[10:0] cnt;
    reg[63:0] yrem;
    wire[31:0] _B = -B;
    wire[63:0] yrem_shift = {yrem[62:0],1'b0};
    wire[63:0] temp = {B,32'h0};
    initial begin
        flag<=0;
        finish<=0;
        cnt<=0;
        yrem<=0;
    end
    always @(posedge i_clk or negedge i_rstn) begin
        if(i_rstn == `rst)begin
            
        end
        else begin
            case(flag)
            0:begin
                if(i_en)begin
                    cnt<=0;
                    flag<=1;
                    finish<=0;
                    yrem <= {32'h0,A};
                end
            end
            1:begin
                yrem <= (yrem_shift[63:32] >= B) ? yrem_shift - temp + 1'b1 : yrem_shift;
                cnt<=cnt+1;
                if(cnt == 31)begin//这里是并行判断
                    flag <=2;
                    finish <= 1;
                end
            end
            2:begin
                flag<=0;
                finish<=0;
            end
            endcase
        end
    end

    assign Y = yrem[31:0];
    assign REM = yrem[63:32];
    assign o_finish = finish;
endmodule
```