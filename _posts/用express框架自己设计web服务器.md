---
title: 用express框架自己设计web服务器
tag: nodejs
categories: 框架
cover: /images/lsp/2.jpg
---



使用nodejs的express框架搭建一台web服务器

<!--more-->

首先要安装express框架
命令行输入
```cmd
npm install -g express
```
等待安装完成即可    
这里是全局安装express框架   
需要设置环境变量NODE_PATH
将express模块路径添加到NODE_PATH中  

另外，你需要简单了解路由的概念
比如：baidu.com/file/2333.jpg
其中 “baidu.com”是host
"/file"指的就是路由，由服务器处理你需要申请访问的路由
"/file/2333.jpg" 需要就是访问的链接



接下来，直接上代码  

```javascript
//导入express模块
var express = require('express');
var bodyParser = require('body-parser');
var url = require('url');
var app = express();

// 创建 application/x-www-form-urlencoded 编码解析
var urlencodedParser = bodyParser.urlencoded({ extended: false })

//这里是服务器处理GET请求的方法
//'*'代表处理任何GET链接
//比如GET baidu.com/2333/ ,"/2333/"就会被该get方法所接受
//同时传递相关参数给回调函数
app.get('*', function (req, res) {

    //解析传入的路由链接，如果传入的链接包含中文，需要使用URI反编码
    var reqPath = decodeURI(url.parse(req.url).pathname);

    //输出GET请求客户端的IP地址，以及链接
    console.log('GET请求地址：' + req.ip.match(/\d+\.\d+\.\d+\.\d+/) + ";链接：" + reqPath);

    //这里做个小示范
    //如果GET /file/的话，就把本地路径下/file/index.html文件发送给客户端
    var filePath;

    if (reqPath.endsWith('/')) {

        filePath = reqPath.substr(1) + 'index.html';

        if (fs.existsSync(filePath)) {
            
            //该方法可直接发送文件，包括文本，图片等...
            res.sendFile(filePath);

        } else {//如果没有该文件

            //该方法直接发送目标数据
            res.send("访问错误！！");
        }
    }
    else {

        filePath = __dirname + reqPath;

        if (fs.existsSync(filePath)) {

            res.sendFile(filePath);
        } else {

            res.send("访问错误！！");
        }
    }
})


//这里是处理POST请求
//'*'表示处理任何POST链接
//比如 POST /ADD/,那么“/ADD/”就会被该方法所接受
//同时传递相关参数给回调函数
app.post('*', urlencodedParser, function (req, res) {

    //解析POST链接
    var reqPath = decodeURI(url.parse(req.url).pathname);

    console.log('POST请求地址：' + req.ip.match(/\d+\.\d+\.\d+\.\d+/) + ";链接：" + reqPath);

    //打印客户端传过来的POST数据，可以是json字符串或者其它什么的
    //比如客户端POST /hello,并发送“hello world”字符串,
    //那么此处就会打印hello world
    console.log(req.body);
    //post请求也需要给客户端一个回应
    res.send("成功！！");
})


//启动服务器
//web服务器需要提供80端口给外部访问
var server = app.listen(80, function () {
    console.log("启动服务器");
    //服务器默认的IP地址就是当前主机的IP地址
    console.log(server.address());
})


```


当然可以可使用专门的GET，POST路由来处理专门的请求

```javascript
app.get('/get',参数省略)
//这里只接受/get路由
//例如 baidu.com/get/ , baidu.com/get/2333.jpg
//不接受 baidu.com/233/


app.post('/post',参数省略)
//这里只接受/post路由
//例如 baidu.com/post/ , baidu.com/post/2333.jpg
//不接受 baidu.com/233/

```

路由也可以是某个文件


```javascript
app.get('/233.jpg',参数省略)
//这里只接受/233.jpg路由
//例如 baidu.com/233.jpg
//不接受 baidu.com/233

```



如果想让客户端访问服务器中一个文件夹里所有的文件
实现一个简单的文件服务器
可以使用 “use”方法  

```javascript
app.use('/public', express.static('DataBase'));
//当客户端提交GET /public/233.jpg请求
//那么服务器会将本地路径下 DataBase/233.jpg发送给客户端

```

有关express框架详细教程可参见[菜鸟教程](https://www.runoob.com/nodejs/nodejs-express-framework.html)

