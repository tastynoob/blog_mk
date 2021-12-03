---
title: FPGA进阶：手撸神经网络
tag: verilog
categories: fpga
---

手撸神经网络
<!--more-->
进过了之前的基础学习，想必大家应该对verilog有了个比较清晰的认识

那么接下来，我们就来撸个神经网络吧！

------

### 首先来介绍一下该神经网络的规格：

__使用IEEE标准的32位浮点数运算__

__网络大小为3\*3\*3__

__激活函数使用relu__

__神经网络为固定结构__

使用组合电路，这意味着只要输入一个数据就能瞬间出来结果

所有的电路模块均为组合逻辑

-----

使用到的基本资源：

浮点运算器，浮点乘法器，浮点加法器，激活函数运算器

### 特别注意：所有的数据类型均为单精度浮点数

-----


__首先是激活函数运算模块__

```
module Act_Func(
	input[31:0] Xi,
	output[31:0] Yo
);

assign Yo = Xi[31] ? 32'd0 : Xi;

endmodule
```

----

__浮点运算单元：__

浮点运算单元比较复杂，这里不贴代码，

```
FPU_ADD(a,b,y);
FPU_MUL(a,b,y);
```

-------


__人工神经元模块：__

```

module Neural(
	input[3*32-1:0] Xi,
	input[3*32-1:0] Ws,//权重
	input[31:0]  B,//偏执
	output[31:0] Yo
);

wire[31:0] o[2:0];

FPU_MUL f1(Xi[0*32+32-1:0*32],Ws[0*32+32-1:0*32],o[0]);
FPU_MUL f2(Xi[1*32+32-1:1*32],Ws[1*32+32-1:1*32],o[1]);
FPU_MUL f3(Xi[2*32+32-1:2*32],Ws[2*32+32-1:2*32],o[2]);


wire[31:0] t[1:0],f;
FPU_ADD f4(o[0],o[1],t[0]);
FPU_ADD f5(o[2],t[0],t[1]);
FPU_ADD f6(B,t[1],f);
Act_Func act(f,Yo);
endmodule

```

------


__神经网络层：__

```
module Layer(
	input[3*32-1:0] Xi,
	input[3*3*32-1:0] Ws,	
	input[3*32-1:0] Bs,
	output[3*32-1:0] Yo
);	

Neural n1(Xi,Ws[0*3*32+3*32-1:0*3*32],Bs[0*32+32-1:0*32],Yo[0*32+32-1:0*32]);
Neural n2(Xi,Ws[1*3*32+3*32-1:1*3*32],Bs[1*32+32-1:1*32],Yo[1*32+32-1:1*32]);
Neural n3(Xi,Ws[2*3*32+3*32-1:2*3*32],Bs[2*32+32-1:2*32],Yo[2*32+32-1:2*32]);

endmodule

```

-----

__神经网络：__

```
//3*3*3的神经网络
module NetWork(
	input[3*32-1:0] Xi,
	output[3*32-1:0] Yo 
);


reg[3*3*3*32-1:0] ws = 
{
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC,
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC,
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC,
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC,
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC,
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC,
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC,
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC,
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC
};

reg[3*3*32-1:0] bs =
{
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC,
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC,
32'h3DCCCCCC,32'h3DCCCCCC,32'h3DCCCCCC
};


wire[3*32-1:0] cen[1:0];
Layer l1(Xi,ws[0*3*3*32+3*3*32-1:0*3*3*32],bs[0*3*32+3*32-1:0*3*32],cen[0]);
Layer l2(cen[0],ws[1*3*3*32+3*3*32-1:1*3*3*32],bs[1*3*32+3*32-1:1*3*32],cen[1]);
Layer l3(cen[1],ws[2*3*3*32+3*3*32-1:2*3*3*32],bs[2*3*32+3*32-1:2*3*32],Yo);

endmodule

```

下面是仿真结果和电脑运行结果：
![1.png](https://i.loli.net/2021/05/29/TkqpgYHiUcmF4S2.png)
![2.png](https://i.loli.net/2021/05/29/z6u2mgJKvPORqxj.png)

嗯，输出结果完全一样

嗯，看着是不是非常nb呢，由于使用了组合电路

因此这个模块只要输入数据就能瞬间输出数据

但由于是固定的，写死了的神经网络，所以它的搭建十分麻烦

完整代码[click me](https://github.com/tastynoob/FPGA_networl.git)

