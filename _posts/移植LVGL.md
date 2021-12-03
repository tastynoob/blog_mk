---
title: 移植LVGL
tag: model
categories: 框架
---

移植LVGL的方法
<!--more-->

首先你要去LVGL的github上下载[lvgl源代码](https://github.com/lvgl/lvgl)  

下载完后解压你会得到如下文件:

![0.png](https://i.loli.net/2021/07/09/J8hZGCDRu29lrBP.png)

其中src文件夹下为LVGL源码，不可修改 

examples下为LVGL与硬件交互的接口模板    

lv_conf_template.h为LVGL设置模板    

lvgl.h为LVGL库调用头文件    

LVGL的移植十分简单  

仅仅只需要提供刷屏函数就能完成lvgl基本的显示功能    

-----

首先把需要的文件全部移到项目里，需要复制一下文件：

```
src文件夹
examples/porting文件夹
lvgl.h和lv_conf_template.h文件
```

![1.png](https://i.loli.net/2021/07/09/oCN42TcZjsgB8zI.png)

接下来是进行基本的配置

首先把lv_conf_template.h文件改名为lv_conf.h

再将lvgl_conf.h宏开启   

![2.png](https://i.loli.net/2021/07/09/ZSt2fpFr6IuY7mB.png)

把0改成1就行了


然后是更改lv_conf.h里面的设置
找到：
```c++

//设置屏幕的像素宽
#define LV_HOR_RES_MAX          (480)
//设置屏幕的像素高
#define LV_VER_RES_MAX          (320)

//设置颜色位宽，默认16位颜色
#define LV_COLOR_DEPTH     16

//设置显存大小
#define LV_DISP_ROT_MAX_BUF  (10U * 1024U)


//设置像素密度，一般设置为60
#define LV_DPI              130     

//设置是否使用GPU，如果没有就设为0关了
#define LV_USE_GPU              1 
//设置是否使用文件系统
#define LV_USE_FILESYSTEM       1

```

目前就这些主要设置，根据自己的实际情况来看  

设置无误后先进行一次编译，如果lvgl设置正确，那么编译应该会通过

接下来就是编写lvgl与硬件交互的底层驱动

打开poting文件夹

把lv_port_disp_template.c改名为lv_port_disp.c

把lv_port_disp_template.h改为lv_port_disp.h

记住修改lv_port_disp.c时同样要修改头文件名

开启这几个文件的宏

修改lv_port_disp.h文件内容如下：

![3.png](https://i.loli.net/2021/07/09/DThcZRdVnFLW2pl.png)

在lv_port_disp.c中找到lv_port_disp_init函数

```c++

/* Example for 1) */
static lv_disp_buf_t draw_buf_dsc_1;
static lv_color_t draw_buf_1[LV_HOR_RES_MAX * 10];                          /*A buffer for 10 rows*/
lv_disp_buf_init(&draw_buf_dsc_1, draw_buf_1, NULL, LV_HOR_RES_MAX * 10);   /*Initialize the display buffer*/

/* Example for 2) */
static lv_disp_buf_t draw_buf_dsc_2;
static lv_color_t draw_buf_2_1[LV_HOR_RES_MAX * 10];                        /*A buffer for 10 rows*/
static lv_color_t draw_buf_2_2[LV_HOR_RES_MAX * 10];                        /*An other buffer for 10 rows*/
lv_disp_buf_init(&draw_buf_dsc_2, draw_buf_2_1, draw_buf_2_2, LV_HOR_RES_MAX * 10);   /*Initialize the display buffer*/

/* Example for 3) */
static lv_disp_buf_t draw_buf_dsc_3;
static lv_color_t draw_buf_3_1[LV_HOR_RES_MAX * LV_VER_RES_MAX];            /*A screen sized buffer*/
static lv_color_t draw_buf_3_2[LV_HOR_RES_MAX * LV_VER_RES_MAX];            /*An other screen sized buffer*/
lv_disp_buf_init(&draw_buf_dsc_3, draw_buf_3_1, draw_buf_3_2, LV_HOR_RES_MAX * LV_VER_RES_MAX);   /*Initialize the display 


```

找到这一段代码

这3个例子是驱动屏幕的方式，选择第一种即可，把其它2种注释掉

在上面的代码下找到这2行

```C++

//显示范围宽度
disp_drv.hor_res = 480;
//显示范围高度
disp_drv.ver_res = 320;

```

在下面找到disp_flush函数，这个函数就是核心的显示屏刷屏函数

```c++

//参数说明：
//disp_drv:不用管
//area:显示范围的x1，y1,x2,y2,坐标
//color_p:像素点，可视为16位整数

static void disp_flush(lv_disp_drv_t * disp_drv, const lv_area_t * area, lv_color_t * color_p)
{
    /*The most simple case (but also the slowest) to put all pixels to the screen one-by-one*/

    int32_t x;
    int32_t y;
    for(y = area->y1; y <= area->y2; y++) {
        for(x = area->x1; x <= area->x2; x++) {
            //这里编写你的绘制一个像素点的函数，但实际上这种效率很低
            /* put_px(x, y, *color_p)*/
            color_p++;
        }
    }
    //如果有类似于这种函数的话，建议用这种，速度更快
    //lcd_draw(area->x1,area->y1,area->x2,area->y2,(uint16_t*)color_p);

    /* IMPORTANT!!!
     * Inform the graphics library that you are ready with the flushing*/
    lv_disp_flush_ready(disp_drv);
}


```

至此，lvgl与显示屏的接口编写完成

接下来做个简单的测试

```c++
#inclue "lvgl.h"


int main(){
    lv_init();
    lv_port_disp_init();
    lv_port_indev_init();

    //创建一个图形按钮
    lv_obj_t* btn = lv_btn_create(lv_scr_act(), NULL);
    //设置按钮大小
    lv_obj_set_pos(btn, 10, 10); 
    //设置按钮位置                           
    lv_obj_set_size(btn, 120, 50);   

    //开启lvgl工作线程，这个函数主要处理lvgl图像绘制和事件响应等任务
    task_run(lv_task_handler);

    while(1){
        //心跳函数，设置为1代表循环一次消耗1ms，用来设置lvgl的事件频率
        //比如lvgl动画，如果在1ms内运行100次lv_tick_inc(1)
        //那么动画速度理论上会快100倍
        lv_tick_inc(1);
        delayms(1);
    }
}

```

如果lvgl能正常工作，那么屏幕上应该会绘制一个按钮


接下来是移植输入设备

与上面一样

在porting下找到lv_port_indev_template.c和lv_port_indev_template.h

然后修改文件名，宏

修改lv_port_indev.h文件内容如下

![4.png](https://i.loli.net/2021/07/09/dPIoROWrMyspJC8.png)


再修改修改lv_port_indev.c里面的代码

lv_port_indev.c里面提供的接口比较多，这里只介绍编写触摸按键接口


找到一下代码


```c++

    /*------------------
     * Button
     * -----------------*/

    /*Initialize your button if you have*/
    button_init();

    /*Register a button input device*/
    lv_indev_drv_init(&indev_drv);
    indev_drv.type = LV_INDEV_TYPE_BUTTON;
    indev_drv.read_cb = button_read;
    indev_button = lv_indev_drv_register(&indev_drv);

    /*Assign buttons to points on the screen*/
    //这里是按钮映射
    //比如按下ID为0的按钮就相当于点击屏幕（10，10）的位置
    static const lv_point_t btn_points[2] = {
            {10, 10},   /*Button 0 -> x:10; y:10*/
            {40, 100},  /*Button 1 -> x:40; y:100*/
    };

    //这里默认使用button接口
    lv_indev_set_button_points(indev_button, btn_points);

```

代码中默认使用button接口，我们只需要把按键映射改一下就行


然后找到按钮的接口


```C++


/*Get ID  (0, 1, 2 ..) of the pressed button*/
//获取按键按下的ID
static int8_t button_get_pressed_id(void)
{
    uint8_t i;

    /*Check to buttons see which is being pressed (assume there are 2 buttons)*/
    for(i = 0; i < 2; i++) {
        /*Return the pressed button's ID*/
        if(button_is_pressed(i)) {
            return i;
        }
    }

    /*No button pressed*/
    return -1;
}

/*Test if `id` button is pressed or not*/
//确认按键是否被按下
static bool button_is_pressed(uint8_t id)
{

    /*Your code comes here*/

    return false;
}



```


这里就按自己的意愿改了



同样再做一个小测试

```c++
#inclue "lvgl.h"


int main(){
    lv_init();
    lv_port_disp_init();
    lv_port_indev_init();

    //创建一个图形按钮
    lv_obj_t* btn = lv_btn_create(lv_scr_act(), NULL);
    //设置按钮大小
    lv_obj_set_pos(btn, 10, 10); 
    //设置按钮位置                           
    lv_obj_set_size(btn, 120, 50);   
    //这里是设置lvgl按钮的事件
    lv_obj_set_event_cb(btn, [](lv_obj_t* btn, lv_event_t event) {
        switch (event) {
            //第一次按下
            case LV_EVENT_PRESSED:
                printf("pressed!\n");
                break;
                //持续按下
            case LV_EVENT_PRESSING:
                printf("pressing!\n");
                break;
                //按钮释放
            case LV_EVENT_CLICKED:
                printf("clicked!\n");
                break;
        }
    });                 /*Assign a call

    //开启lvgl工作线程，这个函数主要处理lvgl图像绘制和事件响应等任务
    task_run(lv_task_handler);

    while(1){
        //心跳函数，设置为1代表循环一次消耗1ms，用来设置lvgl的事件频率
        //比如lvgl动画，如果在1ms内运行100次lv_tick_inc(1)
        //那么动画速度理论上会快100倍
        lv_tick_inc(1);
        delayms(1);
    }
}


````


lvgl同样可以移植文件系统，就跟上面教程一样，这里不加赘述

