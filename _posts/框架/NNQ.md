---
title: NNQ框架
tag: 神经网络
categories: 
    - [框架]
    - [算法]
    - [人工智能]
cover: /images/lsp/3.jpg
---

NNQ是一个用C++设计的算法框架    
包含矩阵,向量,甚至包括神经网络搭建  
只需较少的资源即可完成大部分算法    
推荐用于嵌入式领域

<!--more-->

NNQ框架使用类似于神经网络层级建模的方式来构造神经网络模型       
目前只支持DNN网络的构建     
部分地方仍需优化


github仓库:[nnq](https://github.com/tastynoob/NNQ)

NNQ框架搭建简单的DNN网络:
```
#include <iostream>
#include <time.h>
#include <math.h>
#include <queue>
#include <math.h>
#include <string>
#include <vector>
using namespace std;
#include "NNQ/nnq.hpp"
using namespace nnq;
nnl::Model m = {
        Func::Linear(1,20,0.001),
        Func::Sigmoid(20),
        Func::Linear(20,20,0.001),
        Func::Sigmoid(20),
        Func::Linear(20,1,0.0001),
};
int main() {
    qVec in(1);
    qVec out(1);
    qVec ideal(1);
    qVec loss(1);
    qVec grad(1);
    srand((unsigned)time(NULL));
    mode = 1;
    for(int i = 0; i < 20000; i++) {
        
        for (int j = 0; j < 100; j++) {
            float a = sin(j / 10);
            in[0] = j/10;
            ideal[0] = a;
            out <= m(in);
            Func::square_loss(loss,out, ideal, grad);
            m[grad];
        }
        
        qtype ls =
            loss[0];
        cout << ls << endl;
    }
    mode = 0;
    for (int j = 0; j < 100; j++) {
        int k = abs(rand()) % 100;
        float a = sin(k / 10);
        in[0] = k / 10;
        out <= m(in);
        cout << "real:" << out[0] << " ideal:" << a << endl;
    }
}
```