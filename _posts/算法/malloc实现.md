---
title: malloc实现
tag: 系统
categories: 算法
cover: /images/lsp/4.jpg
---


malloc实现
<!--more-->

下面主要介绍2种malloc实现

第一种是内存的块式管理，第二种是内存的链式管理

首先是贴代码

* 块式管理


```c
#include <stdio.h>

typedef unsigned int u32;

//块式管理，内存块的数量
#define block_num 128
typedef struct {
	int mem[32];//一个块128字节
}block;
#define block_size sizeof(block)
typedef struct {
	block blocks[block_num];
	char table[block_num];
} alloc_mem;

alloc_mem alloc_ptr;

void myalloc_init() {
	sizeof(alloc_mem);
	//初始化表项
	for (int i = 0; i < block_num ; i++) {
		alloc_ptr.table[i] = 0;
	}
}

//分配内存空间
void* myalloc(u32 size) {
	//实际需要分配的连续内存块
	int block_len = (size+8) / block_size + (((size+8) % block_size) ? 1 : 0);

	//从表单中搜索
	for (int thp = 0; thp < block_num - block_len + 1; thp++) {
		int tep = thp + block_len;
		int bvld = 0;//内存块可用
		for (int i = thp; i < tep; i++) {
			bvld |= alloc_ptr.table[i];
		}
		if (!bvld) {//如果bvld==0，代表内存块可用
			//设置占用
			for (int i = thp; i < tep; i++) {
				alloc_ptr.table[i] = 1;
			}
			//将内存块的前4个字节设置为块长度
			*((int*)&alloc_ptr.blocks[thp]) = block_len;
			*(((int*)&alloc_ptr.blocks[thp]) + 1) = thp;//索引指针
			return (void*)(((int*)&alloc_ptr.blocks[thp])+2);

		}//否则继续搜索
	}
	return NULL;//没有搜索到
}

void myfree(void* ptr) {
	int block_len = *((int*)ptr - 2);
	int thp = *((int*)ptr - 1);
	for (int i = thp; i < thp + block_len; i++) {
		alloc_ptr.table[i] = 0;
	}
}



int main() {
	myalloc_init();
	
	int* p = myalloc(10 * sizeof(int));

	for (int i = 0; i < 10; i++) {
		p[i] = i + 1;
	}
	for (int i = 0; i < 10; i++) {
		printf("%d\n", p[i]);
	}
}

```

* 链式管理


```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>


typedef unsigned int u32;

u32* buff;
#define alloc_addr ((int)buff)
#define alloc_size 1024*4


typedef struct {
    int size;//内存块大小
    int avaliable;//是否激活
    void* next;//下一个块
} mem_block;


mem_block* alloc_head;

void myalloc_init() {
    buff = malloc(alloc_size);
    alloc_head = (mem_block*)buff;
    alloc_head->size = 0;
    alloc_head->avaliable = 0;
    alloc_head->next = 0;
}

void* myalloc(u32 sizeofbyte) {
    mem_block* temp = alloc_head;//指向当前块

    if (sizeofbyte == 0) {
        return NULL;
    }

    //实际分配的大小
    int real_size = 4 * (sizeofbyte / 4 + ((sizeofbyte % 4) ? 1 : 0));


    while (1) {

        if (temp->avaliable == 0) { //当前块没有被使用

        alloc_new:

            if (temp->next == 0) { //假如下一个块为空，则分配当前块

                //如果剩余空间不足以分配
                if (((int)temp) + real_size > alloc_addr + alloc_size) {
                    return NULL;
                }

                temp->size = real_size;
                temp->avaliable = 1;
                temp->next = ((char*)temp + sizeof(mem_block)) + real_size;



                //没有超界
                if ((temp->next > buff) && (((int)temp->next) + sizeof(mem_block) < (alloc_addr + alloc_size))) {
                    ((mem_block*)temp->next)->size = 0;
                    ((mem_block*)temp->next)->avaliable = 0;
                    ((mem_block*)temp->next)->next = 0;
                }
                

                break;
            }
            else { //下一个块不为空

               
                if (temp->size >= real_size) { //如果当前块的大小满足要求

                    //如果当前块减去分配的内存，剩余的大小还能够再分配一个小块
                    if (temp->size - real_size > sizeof(mem_block) + 4) {
                        //新块
                        mem_block* temp1 = (mem_block*)(((char*)(temp + 1)) + real_size);
                        temp1->size = temp->size - real_size - sizeof(mem_block);
                        temp1->avaliable = 0;
                        temp1->next = temp->next;

                        temp->next = temp1;
                        temp->size = real_size;
                    }
                    else{
                        real_size=temp->size;
                    }
                    
                    temp->avaliable = 1;
                    break;
                }
                else { //不满足要求 (下一个块不为空并且当前块不满足要求)
                    

                    if (((mem_block*)temp->next)->avaliable == 0) { // 如果下一个块没有激活，则可以分配到当前块

                        temp->size = temp->size + ((mem_block*)temp->next)->size + sizeof(mem_block);//重新分配当前块的大小
                        temp->next = ((mem_block*)temp->next)->next;//重新改变指向下一块的指针

                        goto alloc_new; //重新进行内存分配判断

                    }
                    else {//下一个块已被激活，跳过
                    }
                }
            }
        }

        //如果指针指向正确
        if ((temp->next > buff) && (temp->next < (alloc_addr + alloc_size))) {
            temp = temp->next;
        }
        else {
            return NULL;
        }
    }
    if (temp->size == real_size) {
        return ((char*)temp + sizeof(mem_block));
    }
    return NULL;
}



void myfree(void* ptr) {
    if ((int)ptr >= alloc_addr) {
        mem_block* temp = ((mem_block*)ptr - 1);
        temp->avaliable = 0;
    }
}

int main() {
    myalloc_init();
    //[30,70] 00 000 00
    int* p1 = myalloc(100);
    int* p2 = myalloc(200);
    int* p3 = myalloc(300);
    myfree(p1);
    p1 = myalloc(200);
    int* p4 = myalloc(30);

}
```

-----------

从代码可以看出，块式和链式最根本的区别就在去内存分块机制的不同

块式管理简单方便，但是内存利用率不高
块式管理即简单的将内存分为固定的几块，然后建立一个查找表
根据查找表分配内存

链式管理可以动态分配一定大小的内存块，内存利用率高，但是管理麻烦
链式管理即使用链表动态分配内存和回收，每次分配内存需从头开始查找，如果有未利用内存还需重新分配
较为复杂

以上代码仅供参考