# M扩展

定义了整数乘除法指令扩展（M扩展）的标准。

## 整数乘法
![](../img/rv64_mul.png)

### MUL
```asm
res = GPR[rs1] * GPR[rs2]
GPR[rd] = res[XLEN-1:0]
```
### MULH\[\[S][U]]
```asm
res = 
    MULH: signed(GPR[rs1]) * signed(GPR[rs2])
    MULHU: unsigned(GPR[rs1]) * unsigned(GPR[rs2])
    MULHSU: signed(GPR[rs1]) * unsigned(GPR[rs2])
GPR[rd] = res[2*XLEN-1:XLEN]
```
### MULW
RV64指令，将源寄存器的低32位相乘，得到的64位结果，符号扩展低32位。
```asm
res = GPR[rs1][31:0] * GPR[rs2][31:0]
GPR[rd] = sign-extension(res[31:0])
```
### 指令融合
如果需要同时保存hi值和lo值到寄存器中，则通常使用以下代码：
```asm
MULH[[S]U]  rdh, rs1, rs2; 
MUL         rdl, rs1, rs2;
// or 
MUL         rdl, rs1, rs2;
MULH[[S]U]  rdh, rs1, rs2;
```
1. 前后的源寄存器rs1和rs2顺序相等；
2. 第一个出现的目的寄存器，不能和rs1和rs2相同；
3. 设计关键：存在上述关联的乘法，融合到单个乘法运算中，而不是执行两个单独的乘法运算。



## 整数除法
![](../img/rv64_div.png)

### DIV[U]
用寄存器 (**u**n)signed x[rs1] 的值除以寄存器 (**u**n)signed x[rs2] 的值，向零舍入，把商写入 x[rd] 。

DIV、DIVU分别对应计算结果的 signed 和 unsigned。

### REM[U]
用寄存器 (**u**n)signed x[rs1] 的值除以寄存器 (**u**n)signed x[rs2] 的值，向零舍入，把余数写入 x[rd] 。

REM中余数的符号和被除数rs1相同。

### DIV[U]W / REM[U]W
RV64指令，将源寄存器的(**u**n)signed 低32位相除，得到的32位结果，符号扩展为64位结果，写入 x[rd] 中。

### 指令融合
如果需要同时保存商值和余数值到寄存器中，则通常使用以下代码：
```asm
DIV[U] rdq, rs1, rs2; 
REM[U] rdr, rs1, rs2 
```
1. 前后的源寄存器rs1和rs2顺序相等；
2. 第一个出现的目的寄存器，不能和rs1和rs2相同。

> TIPS: 
> `srl` 可以做除数为 2^i^ 的无符号除法。例如，如果 a2=16(2^4^)，那么 srli t2,a1,4 这条指令和 divu t2,a1,a2 得到的结果相同。
> 同理，`sll` 可以做乘数为 2^i^ 的无符号乘法。例如，如果 a2=16(2^4^)，那么 slli t2,a1,4 这条指令和 mul t2,a1,a2 得到的结
果相同。
> 乘法比移位和加法慢很多，除法比乘法慢很多，所以有时可以利用移位来优化。

### 特殊情况处理
![](../img/rv64_div_except.png)
1. 除以0
商 = 全1
余数 = 被除数

2. 有符号除法溢出
商 = 被除数
余数 = 0

> **注意：没有除零异常。**
要测试除数是否为零，只需要在除法操作之前加入一条用于测试的 beqz 指令。RV32I 不会因为除零操作而 trap，因为极少数程序需要这种行为，而且在那些软件中可以很容易地检查是否除零。当然，除以其它常数永远不需要检查。