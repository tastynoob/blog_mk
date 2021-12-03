---
title: GD32V运行速度慢的解决方法
tag: riscv
categories: 技术
cover: /images/lsp/2.jpg
---

使用longan mcu时，出现了延时1s却实际上感觉延时了5s左右

这个问题实际上是由于官方提供的时钟初始化函数没有执行
<!--more-->

导致核心频率达不到全频108MHz，出现定时器延时错误

我们只需要把该功能开启即可

找到该函数_init()

```

void _init()
{
	SystemInit();

	//ECLIC init
	eclic_init(ECLIC_NUM_INTERRUPTS);
	eclic_mode_enable();

	//printf("After ECLIC mode enabled, the mtvec value is %x \n\n\r", read_csr(mtvec));

	// // It must be NOTED:
	//  //    * In the RISC-V arch, if user mode and PMP supported, then by default if PMP is not configured
	//  //      with valid entries, then user mode cannot access any memory, and cannot execute any instructions.
	//  //    * So if switch to user-mode and still want to continue, then you must configure PMP first
	//pmp_open_all_space();
	//switch_m2u_mode();

    /* Before enter into main, add the cycle/instret disable by default to save power,
    only use them when needed to measure the cycle/instret */
	disable_mcycle_minstret();
}

```

在主函数开头里调用该函数即可