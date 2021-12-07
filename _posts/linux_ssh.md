---
title: vscode上的linux远程开发配置
tag: linux
categories: linux
cover: /images/lsp/1.jpg
---

无
<!--more-->
在利用ssh进行远程开发时

如果要查看远程的图像或者摄像头时

就会无法显示

比如在远程开发opencv时

就会无法查看opencv的窗口

但是我们希望在远程开发时

远程的窗口也能在本地显示

这样就方便了开发和调试

这里我们使用vscode的remote插件进行远程开发

不光可以显示图片，还可以显示和控制图形程序

首先你要在vscode上安装remote ssh和remote x11插件

并能进行免密登录远程服务器

![0](https://i.loli.net/2021/06/30/RrE67LIbk5B8dj9.png)

---------------------

我们来尝试使用windows本地主力机连接远程linux服务器进行远程开发

首先是ssh的配置，确保你拥有openssh并开启了ssh服务

客户端(windows)和服务端(linux)都需要开启

------------

windows下：

windows我们需要生成本地的ssh密匙

打开命令行，输入ssh-keygen,回车

然后他会让你配置密码，我们要免密登录所以直接回车，不配置密码

后面需要回车三次，如果最后输出一段奇怪字符就代表生成成功


打开user/.ssh文件夹，就能看到以下文件

![1](https://i.loli.net/2021/06/30/YlnvXR7O198HoZs.png)

其中id_rsa.pub就是你的公匙，可以用该公匙实现远程免密登录


--------------

linux下：

linux服务端需要允许外部ssh连接

先提前在linux上配置好相关配置

假如你windows远程需要登录服务器上的user用户

在linux上登录user账号

打开终端

输入sudo vim /etc/ssh/sshd_config

![2](https://i.loli.net/2021/06/30/m6MQexHJliO3yFr.png)


找到如图的2项

默认情况下这两项是被注释掉的

将这两项取消注释即可

.ssh/authorized_keys这个是你的外部密匙保存文件

如果你是使用user用户，那么该文件应该就是user/.ssh/authorized_keys

继续终端输入 cd ~/.ssh

输入vim authorized_keys

将在windows上的密匙文件id_rsa.pub里面的内容复制到authorized_keys文件里

保险期间我们一般是先把id_rsa.pub复制到user/.ssh/文件夹下

然后使用cat id_rsa.pub >> authorized_keys

id_rsa.pub里面的内容复制到authorized_keys文件里

然后重启ssh服务service sshd restart

---------------------------------------------


接下来在vscode这边安装好remote ssh和remote x11

打开侧边栏的“远程资源管理器”

![3](https://i.loli.net/2021/06/30/CJVsLhqxg8jimk5.png)


这里我是已经配置好2个远程开发环境了

点击右上角的设置,弹出一个选择框,选择第一个

![4](https://i.loli.net/2021/06/30/3enaGWwhb1HzIYR.png)

其中：
Host是远程服务器的名称，可自定义

HostName是远程服务器IP地址或者网址,必须要填对

Port是远程服务器ssh端口，默认为22

User是你要登录远程服务器的账号，必须要填对，你要使用user账号就填user

另外最下面的3个选项是远程服务器的窗体映射开启

开启这三个选项就能实现远程窗口在本地显示

比如想在远程服务器上显示图片，那么在本地就能生成一个窗口显示远程图片

开启x11还需要安装X server

本地需要下载并安装Xlaunch软件

![5](https://i.loli.net/2021/06/30/LwuNoydx4Szj91T.png)

这个可自行搜索下载

---------------------------------



以上设置配置好后，

![6](https://i.loli.net/2021/06/30/gWXQdR7ZPmM16AD.png)

就可以右键远程资源窗口里的远程服务器进行连接

如图已连接上远程服务器

![7](https://i.loli.net/2021/06/30/YtZFBDRkvues16o.png)

接下来就可以开始愉快的开发了

由于强大的vscode支持，远程开发十分顺滑，比远程桌面更好用


最后再来说一下Xlaunch的用法

打开Xlaunch软件，它会提示你设置窗口界面，直接下一步即可

然后在远程服务器上也要安装remote x11插件

![8](https://i.loli.net/2021/06/30/7wRmH2Eo9MYl1dO.png)

如图选择在SSH:user上安装即可

安装完成后以防万一重启次服务器和vscode

接下来确保x11服务开启，XLaunch开启

在远程服务器上打开一个图片

如果无误，那么本地就会出现一个窗口，里面显示你选择的图片

X11也可以用来显示图形化软件，opencv等窗口，比远程桌面响应更快，延迟更小