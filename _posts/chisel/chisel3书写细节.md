---
title: chisel3书写细节
tag: 
    - [chisel]
categories: fpga
cover: /images/lsp/6.jpg
---

有关chisel3的一些书写细节   
以及chisel生成verilog代码的一些注意事项

<!--more-->

### 首先有必要说的是模块参数化  
可以这样写      

```scala
class dataT extends Bundle {
  val data = UInt(32.W)
  val valid = Bool()
}

class Test[T<:Data](eleT:T) extends Module {
    val io = IO(new Bundle {
        val in = Input(Vec(2, eleT))
        val out = Output(Vec(2, eleT))
    })
    val reg0 = Reg(eleT)
    val reg1 = Reg(eleT)
    reg0 := io.in(0)
    reg1 := io.in(1)
    io.out(0) := reg0
    io.out(1) := reg1
}

object Main extends App {
    (new chisel3.stage.ChiselStage).emitVerilog(new Test(new dataT))
}

```

有助于对模块内硬件类型的参数化,比如在fifo中

### 然后是chisel3的赋值语句

比如下面的语句  

```scala
class Test extends Module{
    val io = IO(new Bundle{
        val in = Input(UInt(32.W))
        val out = Output(UInt(32.W))
    })
    val reg = Reg(UInt(32.W))
    reg:= io.in
    io.out := reg
    io.out := io.in
}
``` 

当对线网类型进行多次赋值时,由于scala本质上是顺序执行的编程语言  
所以后面的语句对线网类型的连接会将前面的赋值的连接覆盖掉    
最后生成的verilog代码如下

```verilog
module Test(
  input         clock,
  input         reset,
  input  [31:0] io_in,
  output [31:0] io_out
);
  assign io_out = io_in; // @[Main.scala 14:12]
endmodule
```
#### 但是当对寄存器类型做多次赋值时

```scala
class Test extends Module{
    val io = IO(new Bundle{
        val in0 = Input(UInt(32.W))
        val in1 = Input(UInt(32.W))
        val out = Output(UInt(32.W))
    })
    val reg = Reg(UInt(32.W))
    reg := io.in0
    reg := io.in1
    io.out := reg
}
```
生成的verilog代码

```verilog
module Test(
  input         clock,
  input         reset,
  input  [31:0] io_in0,
  input  [31:0] io_in1,
  output [31:0] io_out
);
  reg [31:0] reg_; // @[Main.scala 12:18]
  assign io_out = reg_; // @[Main.scala 15:12]
  always @(posedge clock) begin
    reg_ <= io_in1; // @[Main.scala 14:9]
  end
endmodule
```
初步生成的结果和对线网类型的多次赋值的结果是一样的  
#### 但是当我们加上条件判断  

```scala
class Test extends Module{
    val io = IO(new Bundle{
        val in0 = Input(UInt(32.W))
        val in1 = Input(UInt(32.W))
        val en0 = Input(Bool())
        val en1 = Input(Bool())
        val out = Output(UInt(32.W))
    })
    val reg = Reg(UInt(32.W))

    when(io.en0){
        reg := io.in0
    }
    reg := io.in1
    io.out := reg
}
```
生成的verilog代码

```verilog
module Test(
  input         clock,
  input         reset,
  input  [31:0] io_in0,
  input  [31:0] io_in1,
  input         io_en0,
  input         io_en1,
  output [31:0] io_out
);
  reg [31:0] reg_; // @[Main.scala 14:18]
  assign io_out = reg_; // @[Main.scala 20:12]
  always @(posedge clock) begin
    reg_ <= io_in1; // @[Main.scala 19:9]
  end
endmodule
```
很明显，这不是我们想要的verilog逻辑
假如我们调换一下赋值顺序呢

```scala
class Test extends Module{
    val io = IO(new Bundle{
        val in0 = Input(UInt(32.W))
        val in1 = Input(UInt(32.W))
        val en0 = Input(Bool())
        val en1 = Input(Bool())
        val out = Output(UInt(32.W))
    })
    val reg = Reg(UInt(32.W))

    reg := io.in1
    when(io.en0){
        reg := io.in0
    }
    
    io.out := reg
}
```

```verilog
module Test(
  input         clock,
  input         reset,
  input  [31:0] io_in0,
  input  [31:0] io_in1,
  input         io_en0,
  input         io_en1,
  output [31:0] io_out
);
  reg [31:0] reg_; // @[Main.scala 14:18]
  assign io_out = reg_; // @[Main.scala 21:12]
  always @(posedge clock) begin
    if (io_en0) begin // @[Main.scala 17:17]
      reg_ <= io_in0; // @[Main.scala 18:13]
    end else begin
      reg_ <= io_in1; // @[Main.scala 16:9]
    end
  end
endmodule
```

发现逻辑又对了

这是因为chisel对寄存器进行连接时,会按照从上往下的顺序进行连接   
后连接的拥有最大的优先级
假如chisel中出现了对寄存器的多次连接    
生成的verilog代码中会按照chisel从后往前的优先级进行连接
因此在写chisel代码时需要对同一寄存器进行多次赋值连接又不想写太多when语句    
就可以用这个方法来解决这个问题


### 最后就是chisel代码仿真问题      

这里使用最新的chiseltest

典型的测试代码如下

```
class Test extends Module{
    val io = IO(new Bundle{
        val in = Input(UInt(32.W))
        val out = Output(UInt(32.W))
    })
    val reg = Wire(UInt(32.W))
    reg := io.in
    io.out := reg
}

class Test_Test extends AnyFlatSpec with ChiselScalatestTester {
    behavior of "Module"
    it should "work" in {
        //添加如下语句可以生成仿真波形图到本地,可以用gtkwave打开
        test(new Test).withAnnotations(Seq(WriteVcdAnnotation)) { dut =>
            dut.io.in.poke(12.U)
            dut.clock.step(1)
            dut.io.out.expect(12.U)
        }
    }
}
```

注意一下,使用chiseltest无法读取内部寄存器或线网的值,只能读取io接口的值  
因此推荐添加如上代码生成波形图方便测试  








