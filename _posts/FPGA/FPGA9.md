---
title: FPGA学习-8：简单的时序电路 
tag: verilog
categories: fpga
---

简单的时序电路

<!--more-->

经过上一章的了解

组合电路的最大优势就是能直接根据输入进行输出

但其也有很多的缺点：占大量的电路资源，功耗较大，电路固定...

这一次我们来讲基本的时序电路，写一个简单的时序控制电路

再在此基础上将之前的组合电路乘法器改成时序电路


-------------


首先我们要来了解下安路FPGA的时钟信号

安路FPGA的外部24Mhz晶振信号由k14号引脚提供

使用 __“IO Constraint”__ 引脚映射工具

将veilog模块的某一输入端口映射到k14引脚上即可得到24Mhz的时钟信号

先来设计个简单的 __“可变分频器”__

代码如下：

```
module Start(
	input clk,
	input button,	
	output[2:0] led
);


reg[31:0] pll,cnt;
reg[2:0] out;

assign led = ~out;

initial begin
	pll = 32'd24000000;
	cnt = 32'd0;	
	out = 3'b001;
end

always @(posedge clk )begin
	if(cnt <= pll) begin
		cnt = cnt + 32'd1;	 
	end		
	else begin	
		cnt = 32'd0;		
		out = out << 1;	
		if(out == 3'd0)begin		
			out = 3'b001;
		end							
		
	end		
end

integer i=1;

always @(posedge button)begin
	case(i)
		1:begin		
			i=2;		
			pll=32'd12000000;
		end			
		2:begin	
			i=3;		
			pll=32'd6000000;
		end			
		3:begin			
			i=1;		
			pll=32'd24000000;
		end		
	
	endcase

end	

endmodule

```

引脚映射如下：

![1.png](https://i.loli.net/2021/05/29/RzojQeu2HbxyMVg.png)

正常编译再烧录进tang perimer里，led就会交错闪烁

按下user按键，闪烁的频率就会发生改变

这最简单的分频器设计

-----

接下来我们再尝试将上一节中的组合电路单周期乘法器改造

变成时序电路多周期乘法器

额，就不要说什么多此一举了

我们的目的是学习

代码如下

```
module Mul(
	input clk, //时钟信号
	input rst,	//复位信号
	output reg flag,//是否完成信号
	input[7:0] Ai, 
	input[7:0] Bi, 
	output reg[15:0] result
);	

reg[15:0] bi;

initial begin
	bi = {8'd0,Bi};
	flag = 0;	
	result= 0;
end	

integer i=0;

always @(posedge clk or posedge rst) begin
	if(rst == 1)begin
		bi = Bi;
		flag = 0;	
		result = 0;		
		i=0;
	end			
	else if(flag == 0)begin	
		result = result + (Ai[i] ? bi:0);
		bi = bi << 1;		
		i = i + 1;		
		if(i>=8)begin		
			flag = 1;
		end	
	end
end	

endmodule

```

可以看到时序逻辑会比组合逻辑复杂得多

但是也正是因为时序电路的“记忆性”

让cpu这类复杂的运算电路得以实现




