---
title: FPGA-可配置的FIFO模块
tag: verilog
categories: fpga
---

FPGA-可配置的FIFO模块
<!--more-->

```verilog
module FIFO #(
    parameter unitwid = 1,
    parameter unitdpth = 2
)(
    input wire i_clk,
    input wire i_rstn,

    input wire i_wen,
    input wire[unitwid-1:0] i_unitdata,
    output wire o_full,

    input wire i_ren,
    output reg[unitwid-1:0] o_unitdata,
    output wire o_empty
);


    function integer clogb2 (input integer bit_depth);
    begin
        for(clogb2=0; bit_depth>0; clogb2=clogb2+1)
        bit_depth = bit_depth>>1;
    end
    endfunction


    reg[unitwid-1:0] fifo_units[unitdpth-1:0];
    reg[clogb2(unitdpth)-1:0] hptr,eptr,count;

    assign o_full = (count == unitdpth);
    assign o_empty =  ~(|count);

    wire[clogb2(unitdpth)-1:0] hptr_1 = ((hptr == unitdpth) ? 0 : (hptr + 1));
    wire[clogb2(unitdpth)-1:0] eptr_1 = ((eptr == unitdpth) ? 0 : (eptr + 1));


    initial begin
        hptr<=0;
        eptr<=0;
        count<=0;
    end


    always @(posedge i_clk) begin
        if(~i_rstn)begin
            hptr<=0;
            eptr<=0;
            count<=0;
        end
        else begin
            if(i_wen && i_ren)begin//如果同时发生读写
                hptr<=hptr_1;
                eptr<=eptr_1;
            end
            else if(i_wen & (~o_full))begin//只有写
                
                fifo_units[hptr] <= i_unitdata;
                count<=count+1;
                hptr<=hptr_1;
                
            end
            else if(i_ren & (~o_empty))begin//只有读
                count<=count-1;
                eptr<=eptr_1;
                o_unitdata <=fifo_units[eptr];
            end
        end
    end


    
endmodule
```
