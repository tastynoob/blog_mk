---
title: FPGA-RISCV内核入门2
tag: riscv
categories: FPGA
---

FPGA-RISCV内核入门

<!--more-->


首先来了解下基本的rv32i指令集及其编码

![_@1X9___X7GV_V8_3BH~RTT.png](https://i.loli.net/2021/08/29/eJNfPIKxOwtg3vs.png)


![PT9_E@_W@@6~7V_BZVZ70IM.png](https://i.loli.net/2021/08/29/mrlgyDLbKj6i5Bf.png)


根据编码方式的不同大致可以将指令分为6种编码

根据指令功能的不同大致可以分为8种

每个指令的意思都可以在riscv文档中找到

rv32i内核总共有32个通用寄存器

其中寄存器0的值恒为0

![976WK@___K12SOCO6C_KFWJ.png](https://i.loli.net/2021/08/29/TNuUX9YdftcS2I1.png) 

这里也对RISCV指令集、非特权、特权架构不做详细解释

我们只专注于RISCV内核的verilog设计

具体还需去阅读RISCV官方文档

--------

接下来就是简单介绍一下该内核的设计

这里之所以采用五级流水线是因为五级流水线最具代表性和学习性

五级流水线分别是取指、译码、执行、访存、写回

当然，实际开发中可能会做出改进

![FFX6F_DSU_MG57SL_____82.png](https://i.loli.net/2021/09/10/2LOVru7UcyJsFoI.png)







