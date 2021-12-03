---
title: FPGA学习-6：简单的组合电路
tag: verilog
categories: fpga
---
简单的组合电路
<!--more-->

数字电路可分为2大类：

组合电路和时序电路

组合电路的输出只取决于它的输入

并能够在一瞬间完成，与之前状态无关

时序电路则是在时钟控制下有条理的运行

受时钟信号和输入的控制，与之前状态有关

之前的点灯程序就是时序电路

-------

现在我们先从组合电路开始学习

写一个简单的3-8译码器

我们已经了解数字电路的基本组成是逻辑门：

与门，或门，非门

由这3种逻辑门即可组成各种复杂的逻辑电路

组合逻辑电路一般都有个唯一确定的真值表

我们要设计的3-8译码器的真值表如下

| 输入 | 输出 | 
| --- | --- |
| 000 | 00000001|
| 001 | 00000010|
| 010 | 00000100|
| 011 | 00001000|
| 100 | 00010000|
| 101 | 00100000|
| 110 | 01000010|
| 111 | 10000010|


-----


行为级verilog代码如下：

```verilog
module start(
	input[2:0] in,
	output[7:0] out	
);	


assign out = 8'b1 << in;

endmodule
```

仿真波形如下：

![1.png](https://i.loli.net/2021/05/26/CFYgXku9c3BURbi.png)


可以看到，输入输出符合我们设计的真值表

----

但用行为级描述是否过于简单？

那我们来尝试一下门级建模

代码如下

```verilog
module start(
	input[2:0] in,
	output[7:0] out	
);	

wire[2:0] nin;

//非门
not(nin[0],in[0]);
not(nin[1],in[1]);
not(nin[2],in[2]);

//或非门
nor(out[0],in[2],in[1],in[0]);
nor(out[1],in[2],in[1],nin[0]);
nor(out[2],in[2],nin[1],in[0]);
nor(out[3],in[2],nin[1],nin[0]);
nor(out[4],nin[2],in[1],in[0]);
nor(out[5],nin[2],in[1],nin[0]);
nor(out[6],nin[2],nin[1],in[0]);
nor(out[7],nin[2],nin[1],nin[0]);

endmodule
```

仿真波形如下：

![2.png](https://i.loli.net/2021/05/26/ibYlsGPtZxqKI5D.png)

可以看到

真值表与预期一样












