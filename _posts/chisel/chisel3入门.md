---
title: chisel3入门-环境配置
tag: 
    - [chisel]
categories: fpga
cover: /images/lsp/6.jpg
---

chisel3环境配置

<!--more-->


## 介绍

chisel3是一种基于scala编程语言的库,它提供了一个简单的编程接口,可以用来生成verilog代码,以及仿真和调试 
在学习chisel3之前，我们先来学会安装环境 

## 环境配置

chisel3是基于scala编程语言，所以你需要安装scala sdk

同时scala又是基于java虚拟机，因此你也需要安装jdk与jre

jdk或jre的安装不做介绍

scala官网下载地址:https://www.scala-lang.org/download/all.html

我们选择稳定版下载:scala-2.12.15

linux和windows安装软件方法也不做介绍

windows上需要配置环境变量，将scala/bin文件目录添加到PATH中

linux或windows下在终端输入

    
```bash
scala -version
```
如果能正确输出scala版本号就说明你安装成功了

------
接下来我们还需要安装scala项目构建工具sbt

前往sbt官网下载地址:https://www.scala-sbt.org/download.html

按照官方教程即可安装完毕

终端输入
       
```bash
sbt -version
```
(第一次运行sbt可能需要下载一些资源，这个时候可以耐心等待)
如果能正确显示sbt版本号则说明安装无误

-----------

接下来我们还需要安装scala开发软件

虽然可以使用vscode开发，但是vscode的scala插件支持并不友好   
因此还是推荐idea开发    
idea需要安装scala插件才能支持scala      

新建项目如果前面配置都没问题就可以在新建项目里看到scala项目模板 
构建系统选择sbt 
选择好sbt和scala姐创建项目  


## 开始chisel3开发  


创建完成项目后主目录应该会有一个"build.sbt"文件 
文件内容如下

```scala
ThisBuild / version := "0.1.0-SNAPSHOT"//不用管

ThisBuild / scalaVersion := "2.13.8"//scala版本

lazy val root = (project in file("."))
  .settings(
    name := "untitled2"
  )
``` 

就跟java的maven类似 
我们也需要手动添加依赖包

添加chisel3依赖如下 

```scala
ThisBuild / version := "0.1.0-SNAPSHOT"

ThisBuild / scalaVersion := "2.13.8"

val chiselVersion = "3.5.2"
lazy val root = (project in file("."))
  .settings(
    name := "untitled2",
      libraryDependencies += "edu.berkeley.cs" %% "chisel3" % chiselVersion,
      libraryDependencies += "edu.berkeley.cs" %% "chiseltest" % "0.5.2",
      addCompilerPlugin("edu.berkeley.cs" % "chisel3-plugin" % chiselVersion cross CrossVersion.full)
  )
```
此时右上角会出现一个刷新按钮    

点击右上角，重新导入依赖包,等待chisel3下载完毕  

## 开始编写chisel代码

如果chisel3包导入无误，那么我们就可以开始编写chisel代码了

打开src/main/scala/Main.scala文件，添加如下代码

```scala

import chisel3._
import chisel3.stage.ChiselStage
import chisel3.util._

class Test extends Module {
  val io = IO(new Bundle {
    val in = Input(UInt(8.W))
    val out = Output(UInt(8.W))
  })
    io.out := io.in
}

object Main extends App {
  (new ChiselStage).emitVerilog(new Test,Array[String]("--target-dir","Generated"))
}

```

点击object Main 左边的运行按钮即可运行chisel3代码   

运行完成就可以在Generated文件夹下看到生成的Test.v文件了 

那么环境配置就到此结束
