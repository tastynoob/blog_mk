---
title: FPGA学习-9：时序电路之串口通讯模块 
tag: verilog
categories: fpga
---
时序电路之串口通讯模块 

<!--more-->

通信是时序电路的基础之一

要保证信息能够在不同模块直接准确传输

通信协议是保证信息传输的格式要求

那么下面我们就来设计个最简单的

也是最基础的串口通信模块

串口通信时序这里不做介绍

------------

该串口模块的功能描述如下：

波特率9600

停止位1

校验位无

接收字节后将字节加一发送回去

----------

__首先是PLL模块__

PLL模块给串口模块提供稳定的时钟信号

代码如下：

```
module PLL(
	input[31:0] sub,
	input clk,	
	input rst,
	output reg clk_out
);	
reg[31:0] cnt;

initial begin
	cnt = 32'd1;
	clk_out = 0;
end	

always @(posedge clk or negedge rst) begin
	if (rst == 0) begin
		cnt = 32'd1;	
		clk_out = 0;
	end			
	else if(cnt < sub) begin	
		cnt = cnt + 1;
	end			
	else if(cnt >= sub) begin	
		clk_out = ~clk_out;	
		cnt = 32'd1;
	end
end
endmodule
```

-------


__然后是串口模块__

代码如下

```
module USART(
	input clk,	//24Mhz
	input rst,
	input rx,
	output reg tx
);	

reg[7:0] buff;
reg[7:0] rx_shift;
reg[7:0] tx_shift;

initial begin
	tx = 1;
	buff = 0;	
	rx_shift = 0;	
	tx_shift = 0;
end	

wire clk0;
PLL pll(
.sub (32'd1250),
.clk (clk),
.rst (rst),
.clk_out (clk0)
);

integer rx_i = 0;
integer tx_i = 0;
integer rx_flag = 0;
integer tx_flag = 0;
integer send_flag = 0;

always @(posedge clk0 or negedge rst)begin
	if(rst == 0)begin
		rx_i=0;	
		tx_i = 0;
		rx_flag=0;		
		tx_flag = 0;		
		tx = 1;
		buff = 0;	
		rx_shift = 0;		
		tx_shift = 0;
	end		
	else begin
		case(rx_flag)
			0:begin	//起始位
				if(rx==0)begin				
					rx_flag=1;
				end
			end
			1:begin//数据位			
				rx_shift = rx_shift + {rx,7'd0};
				if(rx_i < 7)begin				
					rx_shift = rx_shift >> 1;	
				end	
				rx_i = rx_i + 1;			
				if (rx_i >= 8)begin				
					rx_flag = 2;					
					rx_i = 0;
				end	
			end	
			2:begin//结束位		
				if(rx == 1)begin		
					buff = rx_shift;				
					rx_shift = 0;			
					send_flag = 1;			
					rx_flag = 0;
				end	
			end	
		endcase		
		
		if( send_flag==1 )begin
			case(tx_flag)	
				0:begin		
					tx_shift = buff + 1;
					tx = 0;			
					tx_flag=1;
				end
				1:begin					
					tx = tx_shift[0];	
					tx_shift = tx_shift >> 1;				
					tx_i = tx_i+1;				
					if(tx_i >= 8)begin					
						tx_flag = 2;					
						tx_i = 0;
					end		
				end			
				2:begin						
					tx = 1;			
					tx_flag = 0;	
					send_flag = 0;
				end
			endcase
		end				
	end	
end

endmodule
```

------


管脚映射如下：

![0](https://i.loli.net/2021/06/03/bJHcEdBZDWOugNA.png)


连接USB转串口模块

使用串口调试软件进行测试：

实际使用效果如下;

![1](https://i.loli.net/2021/06/02/5lZkLGe7BVJK6mX.png)

可以看到，我们发送一串十六进制数据

它会自动将数据加一后再返回来

可以证明我们的代码没有问题