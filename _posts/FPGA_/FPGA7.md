---
title: FPGA学习-7：较为复杂的组合电路 上
tag: verilog
categories: fpga
---

FPGA学习-7：较为复杂的组合电路 上

<!--more-->

上一节我们学习了基本的3-8译码器组合电路verilog写法

这一次我们来点有难度的，写一个整型全加器

在此基础上再写一个单周期无符号整型乘法器

------

首先从简单的开始：半加器

### 半加器真值表

| A | B | C |
| --- | --- | --- |
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

真值表可以写为：

__C = A xor B__

可以看到半加器就是2个1位二进制数相加

verilog行为级代码如下：

```
module Half_adder(
    input A,
    input B,
    output Y
);

assign Y = A + B;

endmodule
```

门级代码如下：

```
module Half_adder(
    input A,
    input B,
    output Y
);

xor(Y,A,B);

endmodule
```
 ------

但是要想实现8位，16位，甚至32位的整数加法单单是半加器无法完成的

实际运算过程种也有加法的进位，半加器无法传递进位信息

所以我们需要 __“全加器”__

### 全加器真值表

| A | B | C(上一位进位信号)  | D(当前进位信号) | Y |
| --- | --- | --- | --- | --- |
| 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 | 1 |
| ... | ... | ... | ... | ... |
| 0 | 1 | 1 | 1 | 0 |
| ... | ... | ... | ... | ... |
| 1 | 1 | 1 | 1 | 1 |

真值表可以写为: 

__D = (A and B) or (A and C) or (B and C)__
__Y = (A xor B) xor C__


行为级代码如下:

```
module Adder(
    input A,
    input B,    
	input C,    
	output D,
    output Y
);

assign {D,Y} = A+B+C;

endmodule
```

门级代码如下

```
module Adder(
    input A,
    input B,    
	input C,    
	output D,
    output Y
);

wire t1,t2,t3,t4;
xor(t1,A,B);
xor(Y,t1,C);

and(t2,A,B);
and(t3,A,C);
and(t4,B,C);
or(D,t2,t3,t4);

endmodule
```

由于篇幅原因，下一篇再介绍加法器和乘法器的写法
