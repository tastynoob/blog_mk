---
title: 用纯c语言写一个完整的BP神经网络
tag: 神经网络
categories: 算法
date: 2020-04-22 18:13:00
cover: /images/lsp/0.jpg
---

用纯c语言写一个完整的BP神经网络
<!--more-->

话不多说,直接上代码

所有的说明均在代码中

首先是头文件BPNetWork.h

```c
#ifndef BPNETWORK_H
#define BPNETWORK_H
//所需头文件
#include<math.h>
#include<stdio.h>
#include<stdlib.h>
#include<string.h>


#define f(x) Sigmoid(x)//激活函数设定
#define f_(x) Sigmoidf(x)//导函数

typedef struct {
    double* ws;//权重矩阵
    double* bs;//偏置数组
    double* os;//输出数组
    double* ss;//误差(总误差关于加权和的偏导)
} Layer;
typedef struct {
    int lns;//层数
    int* ns;//每层神经元的数量
    double* is;//神经网络输入
    double* ts;//理想输出
    Layer* las;//神经网络各个层(不包括输入层)
    double ln;//学习率
}BPNetWork;



//创建神经网络
BPNetWork* BPCreate(int* nums, int len,double ln);
//运行一次神经网络
void RunOnce(BPNetWork* network);
//载入训练集
void LoadIn(BPNetWork* network, double* input, double* putout);
//反向传播一次(训练一次)
void TrainOnce(BPNetWork* network);
//输出总误差
double ETotal(BPNetWork* network);

//sigmoid激活函数
#define Sigmoid(x)  (1 / (1 + exp(-(x))))
//sigmoid激活函数的导函数,输入为sigmoid输出
#define Sigmoidf(f)  ((f) * (1 - (f)))
#define Tanh(x) ((2 / (1 + exp(-2 * (x))))-1)
#define Tanhf(f) ((1+(f))*(1-(f)))
#endif
```

然后是程序本体BPNetWork.c

宏定义

```c

#include"BPNetWork.h"


//神经网络的层数
#define LS network->lns

//输入层神经元的数量
#define INNS network->ns[0]

//输入层的第a个输入
#define INS(a) network->is[a-1]

//第a个理想输出
#define TAS(a) network->ts[a-1]

//输出层神经元的数量
#define OUTNS network->ns[LS-1]

//第n层神经元的数量
#define NS(n) network->ns[n-1]

//第n层第a个神经元的第p个权重
#define WF(n,a,p) network->las[n-2].ws[(p-1)+(a-1)*NS(n-1)]

//第n层的第a个神经元的偏置
#define BF(n,a) network->las[n-2].bs[a-1]

//第n层第a个神经元的输出
#define OF(n,a) network->las[n-2].os[a-1]

//第n层第a个神经元的误差
#define SF(n,a) network->las[n-2].ss[a-1]

//学习率
#define LN network->ln
```

BPCreate函数:

```c
BPNetWork* BPCreate(int* nums, int len,double ln)
{
    BPNetWork* network = malloc(sizeof(BPNetWork));
    network->lns = len;
    network->ns = malloc(len * sizeof(int));
    network->ln = ln;
    memcpy(network->ns, nums, len * sizeof(int));
    //
    network->is = malloc(nums[0] * sizeof(double));
    network->las = malloc(sizeof(Layer) * (len - 1));
    network->ts = malloc(sizeof(double) * nums[len - 1]);
    srand(&network);//用networkd的内存地址做为随机数种子
    for (int p = 0; p < len - 1; p++) {
        int lastnum = nums[p];//上一层的神经元数量
        int num = nums[p + 1];//当前层的神经元数量
        network->las[p].bs = malloc(sizeof(double) * num);
        //
        network->las[p].ws = malloc(sizeof(double) * num * lastnum);
        //
        network->las[p].os = malloc(sizeof(double) * num);
        //
        network->las[p].ss = malloc(sizeof(double) * num);
        for (int pp = 0; pp < num; pp++) {
            //这里rand()/2.0的意思是把整数除整数转换为浮点数除整数
            //如果是整数除整数,输出则为带余的商
            network->las[p].bs[pp] = rand() / 2.0 / RAND_MAX;
            for (int ppp = 0; ppp < lastnum; ppp++) {
                network->las[p].ws[ppp + pp * lastnum] = rand() / 2.0 / RAND_MAX;
            }
        }
    }
    return network;
}
```

RunOnce函数:

```c
void RunOnce(BPNetWork* network) {
    //计算输入层到第二层
    for (int a = 1; a <= NS(2); a++) {
        double net = 0;
        double* o = &OF(2,a);//获取第2层的输出值for (int aa = 1; aa <= INNS; aa++) {
            net += INS(aa) * WF(2, a, aa);//INS(aa) * WF(2, a, aa);
        }
        *o = f(net + BF(2,a));
    }
    for (int n = 2; n <= LS-1; n++) {
        for (int a = 1; a <= NS(n + 1); a++) {//下一层的神经网络
            double net = 0;
            double* o = &OF(n+1,a);for (int aa = 1; aa <= NS(n); aa++) {//当前层的神经网络
                double oo = OF(n, aa);
                double* ww = &WF(n + 1, a, aa);
                net += oo * (*ww);
            }
            *o = f(net + BF(n + 1, a));
        }
    }
}
```
TrainOnce函数:

```c
void TrainOnce(BPNetWork* network) {
    //计算输出层的误差函数
    for (int a = 1; a <= OUTNS; a++) {
        double* s = &SF(LS,a);//获取第a个神经元的误差double* b = &BF(LS, a);//获取第a个神经元的偏置
        double o = OF(LS, a);//获取第a个神经元的输出
        *s = (2.0 / OUTNS) * (o - TAS(a))* f_(o);
        *b = *b - LN * (*s);//更新偏置
        //更新权重
        for (int aa = 1; aa <=NS(LS-1) ; aa++) {
            double* w = &WF(LS, a, aa);
            *w = *w - LN * (*s) * OF(LS-1, aa);
        }
    }
    
    //计算隐藏层的误差
    for (int a = LS-1; a > 2; a--) {
        //开始计算第a层每个神经元的误差
        for (int n = 1; n <= NS(a); n++) {//当前层
            double* s = &SF(a, n);//获取第a个神经元的误差
            *s = 0;
            double* b = &BF(a, n);//获取第a个神经元的偏置
            double o = OF(a, n);//获取第a个神经元的输出
            for (int nn = 1; nn <= NS(a+1); nn++) {//下一层
                double lw = WF(a + 1, nn, n);//获取下一层到当前神经元的偏置
                double ls = SF(a + 1, nn);//获取下一层第nn个神经元的误差
                *s += ls * lw * f_(o);
            }
            *b = *b - LN * (*s);//更新偏置
            //更新权重
            for (int nn = 1; nn <= NS(a - 1); nn++) {//上一层
                double* w = &WF(a, n, nn);
                *w = *w - LN * (*s) *OF(a - 1, nn);
            }
        }
    }
    
    //计算第2层的误差函数
    for (int n = 1; n <= NS(2); n++) {//当前层
        double* s = &SF(2, n);//获取第a个神经元的误差
        *s = 0;
        double* b = &BF(2, n);//获取第a个神经元的偏置
        double o = OF(2, n);//获取第a个神经元的输出
        for (int nn = 1; nn <= NS(3); nn++) {//下一层
            double lw = WF(3, nn, n);//获取下一层到当前神经元的偏置
            double ls = SF(3, nn);//获取下一层第nn个神经元的误差
            *s += ls * lw * f_(o);
        }
        *b = *b - LN * (*s);//更新偏置
        //更新权重
        for (int nn = 1; nn <= INNS; nn++) {//上一层
            double* w = &WF(2, n, nn);
            *w = *w - LN * (*s) * INS(nn);
        }
    }
    
}

```


LoadIn函数:
```c
void LoadIn(BPNetWork* network,double* input,double* putout) {
    memcpy(network->is, input, INNS*sizeof(double));
    memcpy(network->ts, putout, OUTNS*sizeof(double));
}
```
ETotal函数:
```c
double ETotal(BPNetWork* network) {
    double val = 0;
    for (int a = 1; a <= OUTNS; a++) {
        val += ((OF(LS, a) - TAS(a)) * (OF(LS, a) - TAS(a))) / OUTNS;
    }
    return val;
}
```
入口函数:
```c
int main() {
    int a[] = { 1,20,20,1 };//4层神经元,数量分别为1,20,20,1
    double in[1] = { 0.9 };//训练样本输入1
    double in1[1] = { 0.1 };//训练样本输入2
    double in2[1] = { 0.5 };//训练样本输入3
    double out[1] = { 0.1 };//理想输出
    //神经网络训练目标:
    //输入任意值,输出0.1
    BPNetWork* network = BPCreate(a, 4, 0.5);
    int c = 1000;//训练1000次
    while (c--) {
        LoadIn(network, in, out);
        RunOnce(network);
        TrainOnce(network);
        LoadIn(network, in1, out);
        RunOnce(network);
        TrainOnce(network);
        LoadIn(network, in2, out);
        RunOnce(network);
        TrainOnce(network);
    }
    //训练完后来一波测试
    double t[1] = { 0.7 };//输入
    double o[1] = { 0.2 };//凑数
    LoadIn(network, t, o);
    RunOnce(network);
    printf("OK\n");
    printf("%g\n", ETotal(network));
    printf("%g", OF(4, 1));
    return 0;
}
```
训练目标:

输入任意数,输出总为0.1

经过1000次训练后的输出:
```
OK
0.0101097//最终误差
0.0994528//输入0.7,输出约为0.1
```