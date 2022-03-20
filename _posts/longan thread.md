---
title: 基于gd32vf的真·多线程系统
tag: [系统,riscv]
categories: 其它
cover: /images/lsp/2.jpg
---


基于gd32vf的真·多线程系统
<!--more-->


目前应用于嵌入式系统的操作系统一般都是

实时多任务操作系统，实际上算是伪多线程

只有当你明确的调用任务delay函数时才会发生调度

以前想设计一个真·多线程操作系统

但是在stm32平台上开发的，由于cm3内核的封闭性，无法明确的修改关键寄存器

但是在riscv上则可以实现，riscv明确了几种中断触发方式

利用自定义中断处理函数可以实现对关键寄存器进行修改，从而达到切换线程的目的

- 下面是gd32v的中断入口

这里使用定时器6作为线程调度时钟，时间设置为1ms中断
````s
  .section      .text.irq	
  .align 2
  .global irq_entry
.weak irq_entry
irq_entry: // -------------> This label will be set to MTVT2 register
  // Allocate the stack space
  
  //保存现场
  SAVE_CONTEXT// Save 16 regs



  //------This special CSR read operation, which is actually use mcause as operand to directly store it to memory
  csrrwi  x0, CSR_PUSHMCAUSE, 17
  //------This special CSR read operation, which is actually use mepc as operand to directly store it to memory
  csrrwi  x0, CSR_PUSHMEPC, 18
  //------This special CSR read operation, which is actually use Msubm as operand to directly store it to memory
  csrrwi  x0, CSR_PUSHMSUBM, 19
 



service_loop:
  //------This special CSR read/write operation, which is actually Claim the CLIC to find its pending highest
  // ID, if the ID is not 0, then automatically enable the mstatus.MIE, and jump to its vector-entry-label, and
  // update the link register 

  csrrw ra, CSR_JALMNXTI, ra 
  
  //RESTORE_CONTEXT_EXCPT_X5

  //恢复现场
  #---- Critical section with interrupts disabled -----------------------
  DISABLE_MIE # Disable interrupts 




  LOAD x5,  19*REGBYTES(sp)
  csrw CSR_MSUBM, x5  
  LOAD x5,  18*REGBYTES(sp)
  csrw CSR_MEPC, x5  
  LOAD x5,  17*REGBYTES(sp)
  csrw CSR_MCAUSE, x5  



  RESTORE_CONTEXT
  // Return to regular code
  mret


.global TIMER6_IRQHandler
//定时器六中断，作为线程切换中断
TIMER6_IRQHandler:
    //保存x18~x27寄存器,s1
  addi sp, sp, -11*REGBYTES
  STORE x27, 0*REGBYTES(sp)
  STORE x26, 1*REGBYTES(sp)
  STORE x25, 2*REGBYTES(sp)
  STORE x24, 3*REGBYTES(sp)
  STORE x23, 4*REGBYTES(sp)
  STORE x22, 5*REGBYTES(sp)
  STORE x21, 6*REGBYTES(sp)
  STORE x20, 7*REGBYTES(sp)
  STORE x19, 8*REGBYTES(sp)
  STORE x18, 9*REGBYTES(sp)
  STORE x9, 10*REGBYTES(sp)



  mv a0,sp //sp
  mv a1,s0 //fp

  addi sp,sp,-4
  sw ra,0(sp)
  call switch_thread
  lw ra,0(sp)
  addi sp,sp,4

  mv sp,x30 //切换sp
  mv s0,x31 //切换fp

  
    //还原x18~x27寄存器,s1
  LOAD x27, 0*REGBYTES(sp)
  LOAD x26, 1*REGBYTES(sp)
  LOAD x25, 2*REGBYTES(sp)
  LOAD x24, 3*REGBYTES(sp)
  LOAD x23, 4*REGBYTES(sp)
  LOAD x22, 5*REGBYTES(sp)
  LOAD x21, 6*REGBYTES(sp)
  LOAD x20, 7*REGBYTES(sp)
  LOAD x19, 8*REGBYTES(sp)
  LOAD x18, 9*REGBYTES(sp)
  LOAD x9, 10*REGBYTES(sp)
  addi sp, sp, 11*REGBYTES


  //清空计数器
  lui	a0,0x40001
  addi	a1,a0,1060
  sw	zero,0(a1)

  //清除中断标志位
  addi	a1,a0,1040
  li	a2,-2
  sw	a2,0(a1)
  ret


````

- 下面是线程切换的关键代码



`````c
#ifndef THREAD_H
#define THREAD_H
#include "config.h"
#include "myalloc.h"
#include "systick.h"

//最大可用线程数
#define max_thread_num 5

typedef void(*Func)();

typedef struct {
    volatile int sp;
    volatile int fp;
    volatile int* stack;
    volatile int wait_tick;//等待tick
    volatile int available;//是否激活
    volatile int pri;//运行优先级
    volatile int re_pri;//重装载优先级,每当一个线程运行完毕就会重置pri
}Thread;

extern volatile int thd_flag;
extern volatile int thd_ptr;
extern Thread thds[max_thread_num];

//线程初始化
void thread_init();
//线程创建
Thread* thread_creat(Func func, int stack_size, int level);
void thread_release(Thread* thd);//线程释放
//线程睡眠
void thread_sleep_tick(int tick);
//线程开启调度
void thread_start();


//在进行一些不可分割操作时，调用该函数进行暂停线程调度
//关闭定时器中断 
//停止计数
#define thd_stop do{\
if (thd_flag == 1)\
{\
TIMER_DMAINTEN(TIMER6) &= (~(uint32_t)TIMER_INT_UP); \
}\
}while (0)


//在进行一些不可分割操作时，调用该函数进行继续线程调度
//开启定时器中断
//继续计数
#define thd_cont do{\
if (thd_flag == 1)\
{ \
TIMER_DMAINTEN(TIMER6) |= (uint32_t)TIMER_INT_UP; \
}\
}while (0)


#endif // !THREAD_H
`````

`````c
#include "thread.h"



int thd_num = 1;
Thread thds[max_thread_num];
//当前线程指针
volatile int thd_ptr = 0;

volatile int thd_flag = 0;




/*
线程调度说明：
每个线程都有一个基础等级,等级越大执行概率越低
基础等级 = level

每个线程运行一次则增加1点等级
其它可用但未执行的线程等级-1




*/

//线程初始化
void thread_init() {
    for (int i = 0;i < max_thread_num;i++) {
        thds[i].fp = 0;
        thds[i].sp = 0;
        thds[i].stack = 0;
        thds[i].wait_tick = 0;
        thds[i].available = 0;
    }
    //主线程,主线程优先级为2
    thds[0].available = 1;
    thds[0].wait_tick = 0;
    thds[0].pri = 2;
    thds[0].re_pri = 2;
}

//线程创建
Thread* thread_creat(Func func, int stack_size, int level) {

    thd_stop;

    for (int i = 0;i < max_thread_num;i++) {

        if (thds[i].available == 0) {

            thds[i].stack = myalloc(stack_size);

            int stack_dep = (stack_size - 1) / 4;

            //设置栈和栈帧
            thds[i].sp = (int)&(thds[i].stack[stack_dep - 31]);
            thds[i].fp = (int)&(thds[i].stack[stack_dep]);

            //msubm所在位置
            thds[i].stack[stack_dep - 31 + 11 + 19] = 64;
            //mecp所在位置
            thds[i].stack[stack_dep - 31 + 11 + 18] = (int)func;
            //mcause所在位置
            thds[i].stack[stack_dep - 31 + 11 + 17] = -1207959478;
            //x1所在位置
            thds[i].stack[stack_dep - 31 + 11 + 0] = (int)func;

            thds[i].available = 1;


            thds[i].pri = level;
            thds[i].re_pri = level;

            return &thds[i];
        }

    }

    thd_cont;

    return NULL;
}
//线程释放
inline void thread_release(Thread* thd) {

    thd_stop;

    thd->available = 0;
    myfree((void*)thd->stack);

    thd_cont;
}

//线程睡眠,并手动触发中断
void thread_sleep_tick(int tick) {


    thds[thd_ptr].wait_tick = tick;

    //等待中断
    asm volatile("wfi");
}




//线程开启调度
void thread_start() {
    thd_flag = 1;
    //清除标志位
    TIMER_INTF(TIMER6) = (~1);


    //开启时钟
    TIMER_CTL0(TIMER6) |= (uint32_t)TIMER_CTL0_CEN;
    //关闭时钟
    //TIMER_CTL0(TIMER6) &= ~(uint32_t)TIMER_CTL0_CEN;
}




volatile int thds_temp[max_thread_num];

//线程切换，由中断程序进行调用
void switch_thread(int sp, int fp) {


    //保存上次中断处的sp、fp
    thds[thd_ptr].sp = sp;
    thds[thd_ptr].fp = fp;

    int temp_head = 0;
    int thd_vld = 0;
    while (1) {
        // int p;
        // asm volatile("mv %0,sp":"=r"(p) : );
        //printf("sp%d\n\n", thds[thd_ptr].wait_tick);


        for (int i = 0;i < max_thread_num;i++) {
            if (thds[i].available) {

                if (thds[i].wait_tick < 1) {
                    thds_temp[temp_head++] = i;//寻找是否有空闲线程,如果有，则保存线程索引
                    thd_vld = 1;
                }
                else {//对于非空闲线程
                    thds[i].wait_tick--;//每个线程的等待时间-1ms
                }
            }
        }

        //printf("dd\n\n");
        if (thd_vld)break;

        delayms(1);//如果没有空闲线程，则等待1ms
    }

    //对可用线程的等级进行从小到大排序
    for (int i = 0;i < temp_head - 1;i++) {
        for (int j = 0;j < temp_head - 1 - i;j++) {
            if (thds[thds_temp[j]].pri > thds[thds_temp[j + 1]].pri) {
                int t = thds_temp[j];
                thds_temp[j] = thds_temp[j + 1];
                thds_temp[j + 1] = t;
            }
            else if (thds[thds_temp[j]].pri == thds[thds_temp[j + 1]].pri) {
                if (mtime_hi() % 2) {//如果有相同的则随机交换
                    int t = thds_temp[j];
                    thds_temp[j] = thds_temp[j + 1];
                    thds_temp[j + 1] = t;
                }

            }
        }
    }

    //取出栈顶索引
    thd_ptr = thds_temp[0];
    //当前线程被执行，重新设置优先级
    thds[thd_ptr].pri = thds[thd_ptr].re_pri;
    //其它未执行的线程优先级增加
    for (int i = 1;i < temp_head;i++) {
        thds[thds_temp[i]].pri--;
    }


    //printf("p%d\n\n", thd_ptr);
    //切换到选中的线程    
    asm volatile("mv x30,%0"::"r"(thds[thd_ptr].sp));
    asm volatile("mv x31,%0"::"r"(thds[thd_ptr].fp));
}
`````


由于只是简单的实验，所以线程调度并没有采用复杂的算法

仅做参考

完整项目[点我](https://github.com/tastynoob/gd32v_thread)