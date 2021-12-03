---
title: FPGA学习-7：较为复杂的组合电路 下
tag: verilog
categories: fpga
---

FPGA学习-7：较为复杂的组合电路 下

<!--more-->

之前我们学会了如何写一个全加器

有了全加器我们就能制造出16位加法器等众多运算器

接下来我们来写个加法器和乘法器

------

简单的8位加法器如下：

```
module Adder8(
    input[7:0] Ai,
    input[7:0] Bi,    
	input Ci,    
	output Do,
    output[7:0] Yo
);

wire[6:0] ds;
//8个全加器
Adder A0(Ai[0],Bi[0],0,ds[0],Yo[0]);
Adder A1(Ai[1],Bi[1],ds[0],ds[1],Yo[1]);
Adder A2(Ai[2],Bi[2],ds[1],ds[2],Yo[2]);
Adder A3(Ai[3],Bi[3],ds[2],ds[3],Yo[3]);
Adder A4(Ai[4],Bi[4],ds[3],ds[4],Yo[4]);
Adder A5(Ai[5],Bi[5],ds[4],ds[5],Yo[5]);
Adder A6(Ai[6],Bi[6],ds[5],ds[6],Yo[6]);
Adder A7(Ai[7],Bi[7],ds[6],Do,Yo[7]);

endmodule
```

如果觉得这样写太麻烦

可以使用 __“批量例化”__

```
module Adder8(
    input[7:0] Ai,
    input[7:0] Bi,    
	input Ci,    
	output Do,
    output[7:0] Yo
);

wire[6:0] ds;
Adder A0(Ai[0],Bi[0],0,ds[0],Yo[0]);

genvar i;
generate
	for(i=1;i<=6;i=i+1) begin
		Adder As(Ai[i],Bi[i],ds[i-1],ds[i],Yo[i]);	
	end
endgenerate	

Adder A7(Ai[7],Bi[7],ds[6],Do,Yo[7]);

endmodule
```

-------------


### 乘法器准备!!!

有了加法器，我们就能实现各种的数学运算

包括乘法

乘法的思想很简单：移位相加

就跟我们拿笔算的思路一样    

制作8位乘法器的输出结果最大有十六位

所以我们需要16位加法器,16位加法器的代码这里不做介绍，照葫芦画瓢即可

由于我们讲的是组合电路

所以接下来我们将制作一个单周期16位乘法器

代码如下：

```
module MUL16(
	input[7:0] Ai,
	input[7:0] Bi,	
	output[15:0] Yo 
);

wire[15:0] a[8:0] ;
 
assign a[0] = Bi[0] ? {8'd0,Ai} : 16'd0;
assign a[1] = Bi[1] ? {7'd0,Ai,1'd0} : 16'd0;
assign a[2] = Bi[2] ? {6'd0,Ai,2'd0} : 16'd0;
assign a[3] = Bi[3] ? {5'd0,Ai,3'd0} : 16'd0;
assign a[4] = Bi[4] ? {4'd0,Ai,4'd0} : 16'd0;
assign a[5] = Bi[5] ? {3'd0,Ai,5'd0} : 16'd0;
assign a[6] = Bi[6] ? {2'd0,Ai,6'd0} : 16'd0;
assign a[7] = Bi[7] ? {1'd0,Ai,7'd0} : 16'd0;

wire d[7:0];
wire[15:0] out[6:0];

Adder16 A0(a[0],16'd0,0,d[0],out[0]);
Adder16 A1(a[1],out[0],d[0],d[1],out[1]);
Adder16 A2(a[2],out[1],d[1],d[2],out[2]);
Adder16 A3(a[3],out[2],d[2],d[3],out[3]);
Adder16 A4(a[4],out[3],d[3],d[4],out[4]);
Adder16 A5(a[5],out[4],d[4],d[5],out[5]);
Adder16 A6(a[6],out[5],d[5],d[6],out[6]);
Adder16 A7(a[7],out[6],d[6],d[7],Yo);


endmodule
```
当然利用verilog语言的特性实际上没必要这样写

但是这样学习FPGA会更加直观的看到电路的逻辑结构

