---
title: RITTER2 Cache实现
tag: 
    - [chisel]
categories: fpga
cover: /images/lsp/6.jpg
---

RITTER2.0内核的Cache实现。包括ICache和DCache

<!--more-->

ICache和DCache均使用chisel3.5设计   
均可配置line数和way数       
采用伪lru替换策略

ICache为只读，DCache为可读可写      
目前均以通过基本的读写测试      

项目地址:[click me](https://github.com/tastynoob/ritter2.x/tree/master/RITTER2) 
ICache 位于IFU文件夹下    
DCache位于Cache文件夹下