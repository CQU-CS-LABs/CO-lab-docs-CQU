# 了解语言

众重大计院学子所周知，我们所学习的硬件语言是 Verilog 。通过纯并行的思想展开了硬件编程的学习。但是本章节想介绍除开 Verilog 之外的其他语言。因为在尝试其他语言的时候，会存在更简洁、高效的使用，不用再为连线、模块烦恼啦。

## SystemVerilog[^blog_sv]

硬综语言选择中，强烈推荐 SystemVerilog，简单上手，且减少了很多重复性劳动（如接口操作、结构体设计等）。

### 模块连接
接口（`interface`），定义在`interface`和`endinterface`之间，独立于模块。好处总结：
- 接口代码可以复用
- 更改模块的接口时，只需要更改interface内部，非常实用。

```Verilog
interface if_id_bus;// 定义接口
wire read_request, read_grant;
wire [7:0]address, data;
endinterface

module RAM(
    if_id_bus io, // 使用接口
    input clk
);
    //可以使用io.read_request引用接口中的一个信号
    ...
endmodule


module CPU(
    if_id_bus io, 
    input clk
);
    ...
endmodule

module top;
reg clk = 0;
if_id_bus a; // 实例接口
//将接口连接到模块实例
RAM mem(a,clk);
CPU cpu(a,clk);
endmodule
```

为了在`interface`中表现出 input/output 属性以保证连线方向的正确性，可以使用 `modport` 将 `interface` 信号进行分组打包。

```Verilog
interface module_if(input clk);
    logic rst_n,
    logic port_a_0 ;
    logic port_a_1 ;
    logic port_b_0 ;
    logic port_b_1 ;

    modport A(
        input rst_n ,
        input port_a_0 ,
        input port_a_1 ,
        output port_b_0 ,
        output port_b_1
    );

    modport B(
        input rst_n ,
        output port_a_0 ,
        output port_a_1 ,
        input port_b_0 ,
        input port_b_1 
    );

endinterface

module module_a( module_if.A U_IF);
    ......
endmodule

module module_b( module_if.B U_IF);
    ......
endmodule

module top();
    logic clk ;
    always #10 clk = ~clk ;
    module_if U_IF(clk);
    initial begin
        U_IF.rst_n = 0 ;
        #50;
        U_IF.rst_n = 1 ;
    end
    module_a U_A(U_IF);  //注意此处不需要再声明modport的名字了
    module_b U_B(U_IF);  //注意此处不需要再声明modport的名字了
endmodule
```
### 数据类型
- `char`: 一个两态的有符号变量，它与C语言中的char数据类型相同，可以是一个8位整数（ASCII）或short int（Unicode）；
- `int`: 一个两态的有符号变量，它与C语言中的int数据类型相似，但被精确地定义成32位；
- `shortint`: 一个两态的有符号变量，被精确地定义成16位；
- `longint`: 一个两态的有符号变量，它与C语言中的long数据类型相似，但被精确地定义成64位；
- `byte`: 一个两态的有符号变量，被精确地定义成8位；
- `bit`: 一个两态的可以具有任意向量宽度的无符号数据类型，可以用来替代Verilog的reg数据类型；
- `logic`: 一个四态的可以具有任意向量宽度的无符号数据类型，可以用来替代Verilog的线网或reg数据类型，但具有某些限制；
- `shortreal`: 一个两态的单精度浮点变量，与C语言的float类型相同；
- `void`: 表示没有值，可以定义成一个函数的返回值，与C语言中的含义相同。
> System Verilog 通过unsigned关键字将有符号数据类型显式地声明成有无符号数据类型。例如: 
> ```Verilog
> unsigned int i;
> ```

### 用户定义类型
关键词： typedef
```Verilog
typedef unsigned int uint;
uint a, b;
```

### 枚举类型
关键词： enum

一个枚举类型具有一组被命名的值。缺省情况下，值从初始值0开始递增，但是我们可以显式地指定初始值。
```Verilog
enum {red,yellow, green} RGB;
enum {WAIT=2’b01, LOAD, DONE} states;
```

### 结构体和联合体
SystemVerilog增加了结构体和联合体，它们的声明语法类似于C。
结构体可以作为一个整体传递到函数或任务，也可以从函数或任务传递过来，也可以作为模块端口进行传递。和`interface`相似。
```Verilog
typedef struct {
  reg [15:0] opcode;
  reg [23:0] addr;
} A;

typedef union {
  int I;
  shortreal f;
}B;

A IR;
B N;
IR.opcode = 1; // 设置IR变量中的opcode域
IR = {5,200}; // 一个结构体可以使用值的级联来完整地赋值
N.f = 0.0; // 将N设置成浮点数的值
```

#### 压缩和非压缩的区别
[Systemverilog 压缩数组 packed_小羊肖恩想的博客-CSDN博客_压缩数组](https://blog.csdn.net/qq_42043804/article/details/122529744)

### 常量
在Verilog中有三种特性类型的常量：parameter、specparam和localparam。
而在SystemVerilog中，允许使用 const 关键字声明常量。例如：

### 字母值
- 一个字母值的所有位均可以使用'0、'1、'z或'x作相同的填充。这就允许填充一个任意宽度的向量，而无需显式地指定向量的宽度，例如：
    ```Verilog
    bit [63:0] data;
    data = '1; //将data的所有位设置成1
    data = '0; //将data的所有位设置成0,初始化常用
    ```
-  一个字符串可以赋值成一个字符数组，象C语言一样加入一个空结束符。如果尺寸不同，和C中一样进行左调整，例如
    ```Verilog
    char foo[0:12] = "hello worldn";
    ```
- 数组可以使用类似于C初始化的语法赋值成字符值，但它还允许复制操作符。括号的嵌套必须精确地匹配数组的维数
    ```Verilog
    int n[1: 2] [1:3] = '{'{0, 1, 2}, '{3{4}}};
    ```

### 强制类型转换
SystemVerilog 通过使用<type>'操作符提供了数据类型的强制转换功能。这种强制转换可以转换成任意类型，包括用户定义的类型。
```Verilog
int'(2.0 *3.0) // 将结果转换为int类型
mytype'(foo) // 将foo转换为mytype类型
17'(x- 2) // 将结果转换为17位宽度
signed'(x) // 将x转换为有符号值
```

### 新增操作符
- ++和--：递增和递减操作符；
- +=、-=、*=、/=、%=、&=、^=、|=、<<=、>>=、<<<=和>>>=赋值操作符；

### 唯一性和优先级
关键词： unique, priority
SystemVerilog能够显式地指明什么时候一条决定语句的分支是唯一的，或者什么时候需要计算优先级。工具使用这些信息来检查if或case语句是否正确建模了期望的逻辑。

```Verilog
bit [2:0]a;

unique if((a==0) || (a==1)) y= in1;
else if (a==2) y=in2;
else if (a==4) y=in3; // 值3、5、6、7会引起一个警告
 
priority if (a[2:1]==0) y = in1; // a是0或1
else if (a[2]==0) y = in2; // a是2或3
else y = in3; // 如果a为其他的值

unique case (a)
  0, 1: y = in1;
  2: y = in2;
  4: y = in3;
endcase // 值3、5、6、7会引起一个警告

prioritycasez(a)
  2'b00?: y = in1; // a是0或1
  2'b0?? : y = in2; // a是2或3
  default : y = in3; //如果a为其他的值
endcase
```

### 新的过程
Verilog使用always过程可以表示时序逻辑、组合逻辑和锁存逻辑的RTL模型。综合工具和其它软件工具必须根据过程起始处的事件控制列表以及过程内的语句来推断always过程的意图。这种推断会导致仿真结果和综合结果之间的不一致。SystemVerilog增加了三个新的过程来显式地指示逻辑的意图。
- always_ff：表示时序逻辑的过程；
- always_comb：表示组合逻辑的过程；
- always_latch：表示锁存逻辑的过程。

### 系统函数
提到的系统函数，部分在 Verilog 中也适用，这里列举一些常用的：
- `$bit(x)` 获取保存 x 需要的硬件位的数目
- `$clog2(x)` 获取 x 的2的对数

### 其他
1. `>>` 和 `>>>` 的区别
`>>` 是二进制逻辑移位，`>>>` 是二进制算术移位。
- 算术右移（ `>>>` ）：向右移指定的位数，如果表达式是带符号的，则用符号位的值填充，否则用零填充，
- 算术左移（ `<<<` ）：向左移指定的位数，用零填充。
- 逻辑移位（ `<<` ，`>>`）：始终用零填充空位位置。

## Chisel

Chisel (The Constructing Hardware in a Scala Embedded Language) ，适合硬件敏捷开发使用，在硬件语言中引入了面向对象属性的设计，可定义参数化模块并生成。Chisel 的学习成本会比 System Verilog 高，但是后续开发效率会比 System Verilog 快不少。像 [rocket-chip](https://github.com/chipsalliance/rocket-chip/)、[boom](https://github.com/riscv-boom/riscv-boom)、[香山](https://github.com/OpenXiangShan/XiangShan/) 等开源处理器，均为 Chisel 开发。

!!! info 
    推荐 Chisel 选手选择 IDEA 编辑器，因为据目前了解仅 IDEA 支持 Scala 语法的跳转。


本教程中只推荐 Chisel 学习资料，有兴趣的同学可自行学习。

- ★★★★★ [Github Chisel Bootcamp 新手训练营](https://github.com/freechipsproject/chisel-bootcamp)
- ★★★★★ [CSDN Chisel 中文教程](https://blog.csdn.net/qq_34291505/article/details/86744581)
- ★★★★ [Chisel 小抄](https://chisel.eecs.berkeley.edu/doc/chisel-cheatsheet3.pdf)
- ★★★ [Chisel op总结](https://www.chisel-lang.org/chisel3/docs/explanations/operators.html)
- ★★★ [Chisel API](https://chisel.eecs.berkeley.edu/api/latest/index.html)
- ★★★ [scala文档](https://docs.scala-lang.org/scala3/book/introduction.html)


## SpinalHDL

> 只在龙芯杯培训中，听大佬推荐过 T_T。

[^blog_sv]: [System Verilog的概念以及与verilog的对比_gtatcs的博客-CSDN博客_systemverilog和verilog的区别](https://blog.csdn.net/gtatcs/article/details/8970489)
