---
title: FPGA-RISCV内核入门4
tag: riscv
categories: 技术
---

接下来我们就开始设计一个简单的取指单元

<!--more-->

首先，取指就是从储存器中取出指令

根据RISCV的指令集特征

我们将指令指针（pc寄存器）设置在取指段

为了保证取指正确，我们还要加入一个简单的握手协议

------

第一个周期取指段发出取指信息

第二个周期储存器读取信息发出指令，同时取指段发出第二个取指信息

第三个周期取指段读取指令，同时取指段发出第3个取指信息，储存器读取第二个信息

-----

大致时序如下，这里没有考虑取指异常等问题

默认为不会出现异常

![F0K_Z_6HSNR7Q___C7BSTKN.png](https://i.loli.net/2021/09/13/WAFXdLwt42D5kbz.png)

取指段如下设计

```verilog

//取指模块
module IF (
    //时钟信号
    input wire i_clk,
    //复位信号
    input wire i_rst_n,

/*分割线*/
    //pc写
    input wire i_pc_we,
    input wire[31:0] i_pc_wdata,

    input wire i_pipe_stop,
/*分割线*/

//下面连接ibus总线

    //ibus读取地址
    output wire[31:0] o_ibus_addr,
    //ibus读取数据
    input wire[31:0] i_ibus_data,
    //ibus请求信号
    output reg o_ibus_req,
    //ibus响应信号
    input wire i_ibus_rsp,

/*分割线*/

//下面是传递下一级流水线

    //传递给下级流水线的ibus地址
    output wire[31:0] o_iaddr,


    //传递给下级流水线的ibus数据
    output wire[31:0] o_idata


);

    reg[31:0] pc;//pc寄存器



    initial begin
        pc = 32'h0;

        o_ibus_req = 0;
    end


    wire valid = i_ibus_rsp & (~i_pipe_stop) ;

    
    always @(posedge i_clk or negedge i_rst_n) begin

        if(i_rst_n == `rst)begin//复位
            pc <= 32'h0;
            o_ibus_req <= 0;
        end
        else if(valid)begin
            pc <= (i_pc_we) ? i_pc_wdata : (pc+4);
            

        end
        else if(i_pipe_stop) begin

        end

        //发送ibus访问请求
        o_ibus_req <= 1;
    end


    //将pc打一拍发送给下级流水线
    //gen_dff#(32)pc_dff(i_clk,i_rst_n,valid,pc,o_iaddr);


    assign o_iaddr = pc;

    assign o_ibus_addr = pc;

    assign o_idata = (i_ibus_rsp & (~i_pc_we)) ?  i_ibus_data : `inst_nop;

    
endmodule


//取指与译码模块之间的隔离模块
module IF_ID (
    input wire i_clk,
    input wire i_rst_n,
    
/*分割线*/

    //流水线暂停
    input wire i_pipe_stop,
    //流水线冲刷
    input wire i_pipe_flush,

    
/*分割线*/
    //上级输入的指令地址
    input wire[31:0] i_iaddr,
    
    //ibus输出的指令数据,
    input wire[31:0] i_idata,

/*分割线*/

    //传递给下级的指令地址
    output wire[31:0] o_iaddr,
    //输出的指令数据
    output wire[31:0] o_idata

);


    wire en =  (~i_pipe_stop | i_pipe_flush);


    reg[31:0] iaddr,idata;

    initial begin
        iaddr<=0;
        idata<=0;
    end

    assign o_iaddr = (i_pipe_flush == `en) ? 32'h0 : iaddr;
    assign o_idata = (i_pipe_flush == `en) ? `inst_nop : idata;


    always @(posedge i_clk or negedge i_rst_n) begin
        if(i_rst_n == `rst)begin
            iaddr <= 32'b0;
            idata <= `inst_nop;
        end
        else if(en == `en)begin

            iaddr <=  i_iaddr;

            idata <=  i_idata;
           
        end
    end

endmodule


```


这里为了简便起见，并没有过多修饰，但也能够使用