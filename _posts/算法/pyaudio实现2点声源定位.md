---
title: pyaudio实现2点声源定位
tag: 
    - [声学]
    - [信号处理]
categories: 算法
cover: /images/lsp/9.jpg
---

使用pyaudio和numpy设计的2点声源定位算法

<!--more-->

不墨迹直接上代码
代码如下

```python
import pyaudio
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

#2个麦克风
nmicro=2

#2个麦克风阵元的坐标,以麦克风a为原点,b为x轴负轴,单位:米
#这里使用的是电脑麦克风,2个麦克风间距约为2分米
pos = np.array([0,-0.2])

print(pos)

#PyAudio的信号采集参数
CHUNK = 4800
FORMAT = pyaudio.paInt16
CHANNELS = 2
RATE = 48000
p = pyaudio.PyAudio()
stream = p.open(format=FORMAT,
                channels=CHANNELS,
                rate=RATE,
                input=True,
                frames_per_buffer=CHUNK)


#遍历的x，假设z为固定深度1m
X_STEP=100
x = np.linspace(-0.8, 0.8, X_STEP)

A_STEP=100
p = np.zeros(x.shape[0])  # 声强谱矩阵
while True:
    data = stream.read(CHUNK,exception_on_overflow=False)#1600*2*2

    data = np.frombuffer(data, dtype=np.short)

    data = data.reshape(CHUNK, 2)[:, :2].T#划分左右声道,shape:(2,1600)

    #傅里叶变换，在频域进行检测
    data_n = np.fft.fft(data)/data.shape[1]  # [2,1600]

    data_n = data_n[:, :data.shape[1]//2] #取前一半频率

    data_n[:, 1:] *= 2 #将频率范围内的频率幅值翻倍

    #宽带处理，对于50个不同的频率都进行计算
    #r存储每个频率下对应信号的R矩阵
    r = np.zeros((A_STEP, nmicro,nmicro), dtype=complex)

    for fi in range(1, A_STEP+1):
        #计算每个频率下的R矩阵
        #自相关函数
        rr = np.dot(data_n[:, fi*10-10:fi*10+10],
                    data_n[:, fi*10-10:fi*10+10].T.conjugate())/nmicro
        r[fi-1, ...] = np.linalg.inv(rr)

    #MVDR搜索过程
    for i in range(x.shape[0]):
        dm = np.sqrt(x[i]**2 + 1)
        delta_dn = pos*x[i]/dm
        #遍历角度
        for fi in range(1, A_STEP+1):
            #计算导向向量
            a = np.exp(-1j*2*np.pi*fi*100*delta_dn/340)
            p[i] = p[i] +  1 / np.abs(np.dot(np.dot(a.conjugate(), r[fi-1]), a))#计算每个频率下的声强谱

    p /= np.max(p)
    #获取前5个最大值
    p_max = np.argsort(p)[-5:]
    #输出声源所在的方向
    print(np.average(p_max))
    #以波形图显示声强谱
    plt.clf()
    plt.plot(x, np.abs(p))
    plt.xlabel('x')
    plt.ylabel('p')
    plt.pause(0.01)

```

有时间再仔细讲讲算法推导
