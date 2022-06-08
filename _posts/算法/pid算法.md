---
title: PID算法
tag: 
    - [技术]
    - [信号处理]
categories: 算法
cover: /images/lsp/7.jpg
---

PID算法
<!--more-->

在系统控制领域，pid算法算是最常见的一种控制算法

由于PID算法比较简单

这里直接贴python代码


``` python
#模拟pid控制器
kp = 0.05
ki = 0.01
kd = 0.01

#一次pid迭代
last_error = 0
integral = 0
def pid_iteration(pv, set_point):
    global last_error
    global integral
    error = set_point - pv
    integral = integral + error
    derivative = error - last_error
    output = kp * error + ki * integral + kd * derivative
    last_error = error
    return output

start = 0
point = 100
for i in range(0,100):
    start = start + pid_iteration(start, point)
    print(start)
```








