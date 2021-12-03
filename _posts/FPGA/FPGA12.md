---
title: FPGA-10：设计个简单的cpu(真的简单！)
tag: verilog
categories: fpga
---

编写一个简单的cpu
<!--more-->

经过了之前的学习

想必各位对verilog应该有了基本的基础

那么，接下来，我们就来造cpu吧！

我们将写一个简单的单周期cpu

* 该cpu有一下特点：
  * 32位架构
  * 单周期执行
  * 简洁实用
  * 32位定长指令
  * 有手就行

我称之为 “ant” 内核

就跟蚂蚁一样，“功能弱小”，但也什么能干

我也特地为该cpu编写了个汇编器

包括使用python编写的bin转txt工具

连接如下：

[click me](https://github.com/tastynoob/FPGA_verilog_easy_cpu)

下载该项目

即可得到5个文件
```
cpu.v: ant内核核心文件

test.v : ant内核仿真文件

ant-asm.exe: ant汇编器

binTotxt.py:将bin文件转换成verilog可读取的储存器填充文件

demo.ant:ant汇编例程
```

---------------

下面是寄存器说明及指令集：

```
寄存器：
r0~r8 共16个32位寄存器，均可可读可写，
r0~r14 通用寄存器
r15 pc寄存器

指令集:(总共14条)
wh r0,num 写r0寄存器的高16位  {8{指令},4{寄存器id},16{常数}，4{无意义}}
wl r0,num 写r0寄存器的低16位  {8{指令},4{寄存器id},16{常数}，4{无意义}}

add r0,r1,r2 : r0 = r1 + r2 //整数加法 {8{指令}，4{r0寄存器id},4{r1寄存器id},4{r2寄存器id},12{无意义}}
sub r0,r1,r2 : r0 = r1 - r2 //整数减法 {8{指令}，4{r0寄存器id},4{r1寄存器id},4{r2寄存器id},12{无意义}}
or r0,r1,r2 : r0 = r1 | r2    {8{指令}，4{r0寄存器id},4{r1寄存器id},4{r2寄存器id},12{无意义}}
and r0,r1,r2 : r0 = r1 & r2   {8{指令}，4{r0寄存器id},4{r1寄存器id},4{r2寄存器id},12{无意义}}
not r0,r1 : r0 = ~r1          {8{指令}，4{r0寄存器id},4{r1寄存器id},16{无意义}}
sl r0,r1,r2 r0 = r1 << r2     {8{指令}，4{r0寄存器id},4{r1寄存器id},4{r2寄存器id},12{无意义}}
sr r0,r1,r2 : r0 = r1 >> r2   {8{指令}，4{r0寄存器id},4{r1寄存器id},4{r2寄存器id},12{无意义}}
mt r0,r1,r2 : r0 = r1 > r2 //比大小 返回0或1 {8{指令}，4{r0寄存器id},4{r1寄存器id},16{无意义}}
lt r0,r1,r2 : r0 = r1 < r2//同上 {8{指令}，4{r0寄存器id},4{r1寄存器id},16{无意义}}

rd r0,r1 : r1 = *r0//读内存   {8{指令}，4{r0寄存器id},4{r1寄存器id},16{无意义}}
wd r0,r1 : *r0 = r1//写内存   {8{指令}，4{r0寄存器id},4{r1寄存器id},16{无意义}}

if r0 :如果r0为非0，则跳过下一条指令
```


虽然指令只有短短14条，但也几乎能够完成所有事了

--------

接下来我们来写个简单的1+2+3+...+100的程序

c代码如下：

```
int main() {
	int all = 0;
	int i = 1;
	int j = 101;
	int k = 0;

l1:
	k = i < j;
	if (!k)
		goto l2;
	else
		all = all + i;
	i = i + 1;
	goto l1;
l2:

	printf("%d", all);
}
```

c代码对应的汇编代码
汇编代码如下：

```
//0起始地址
wl r0,0 //恒为0
wh r0,0
wl r1,1 //恒为1
wh r1,0
//累加
wl r2,0
wh r2,0
//i
wl r3,1 
wh r3,0
//101
wl r4,101 
wh r4,0
//k
wl r5,0
wh r5,0
//地址16
wl r6,16
wh r6,0
//地址22
wl r7,22
wh r7,0
//地址16：
lt r5,r3,r4
if r5//如果r3小于r4
add r15,r7,r0
add r2, r2, r3
add r3,r3,r1
add r15,r6,r0
//地址22：
wd r0,r2

```

将该汇编代码保存进"demo.ant"文件中

输入命令：

```
ant-asm demo.ant

```

即可得到

out.bin可执行文件

再执行binTotxt.py脚本（注意更改里面的路径）

得到out.list填充文件

仿真test.v文件，将out.list内容填充进rom

运行

仿真效果如下：

![0.png](https://i.loli.net/2021/06/09/NOub7LhTJpSrawx.png)

可以看到，正确的算出了5050这个值

说明代码和ant内核是没有什么问题的