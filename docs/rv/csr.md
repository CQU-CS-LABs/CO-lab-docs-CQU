# RISC-V 特权级介绍

相信大家已经在操作系统课程中对特权级有了基础的了解，我们在实现特权级还需要实现特权指令以及CSR寄存器，以及异常处理的逻辑。

这一部分是对[RISC-V Privileged Spec](https://riscv.org/technical/specifications/)中我们实验需要用到的部分进行精炼，精炼到只需要实现一些简单的逻辑，并最终能完成运行Linux NOMMU的目标，同学们课后有时间建议直接阅读英文Spec。


## 特权等级

RISC-V分为以下几个特权等级：

| 等级编号 | 名字             | 简称 | 实现难度 | 目标建议                             |
| -------- | ---------------- | ---- | -------- | ------------------------------------ |
| 0        | User/Application | U    | ☆        | 想要运行U-Boot+Linux NOMMU的同学实现 |
| 1        | Supervisor       | S    | ☆☆☆      | （课程期间很难做出来）               |
| 2        | Hypervisor       | H    | ☆☆☆☆☆    | （课程期间几乎不可能做出来）         |
| 3        | Machine          | M    |          | 必须实现，且默认在M-Mode运行         |

在后文中，Machine Mode中文名为“M态”，User Mode中文名为“U态”。

我们推荐同学们实现M态+U态，相比纯M态并没有增加几行代码。

### 特权等级的描述

RISC-V不同于MIPS，当前特权级状态没有显式保存在CSR中，也不可由所有CSR的取指推导，因此你在设计你的CPU特权管理功能时，请务必增加一个2-bit的寄存器（硬件上的寄存器，ISA级不可见）用于保存当前特权级信息，且在reset时将值设置为`11`（Machine Mode），Machine Mode下可以通过`MRET`这条Machine Mode下的异常返回指令来切换特权级。

## CSR寄存器概述

实现单核运行Linux NOMMU目标，我们需要实现以下CSR寄存器（其实实现异常处理也只有这些寄存器：

| 地址（16进制） | 地址（2进制） | 名称       | 用途                                | 权限 | 简单实现 |
| -------------- | ------------- | ---------- | ----------------------------------- | ---- | -------- |
| 0xc00          | 110000000000  | cycle      | 当前处理器运行周期数                | URO  |          |
| 0xc02          | 110000000010  | instret    | 当前处理器提交指令数                | URO  |          |
| 0xf11          | 111100010001  | mvendorid  | 厂商ID                              | MRO  | 只读0    |
| 0xf12          | 111100010010  | marchid    | 架构ID                              | MRO  | 只读0    |
| 0xf13          | 111100010011  | mimplid    | 实现ID                              | MRO  | 只读0    |
| 0xf14          | 111100010100  | mhartid    | Hart ID                             | MRO  | 只读0    |
| 0xf15          | 111100010101  | mconfigptr | M态配置指针                         | MRO  | 只读0    |
| 0x300          | 001100000000  | mstatus    | M态状态配置                         | MRW  |          |
| 0x301          | 001100000001  | misa       | M态指令集架构与扩展                 | MRW  | 只读（具体多少取决于你的CPU设计） |
| 0x304          | 001100000100  | mie        | M态中断使能                         | MRW  |          |
| 0x305          | 001100000101  | mtvec      | M态Trap向量                         | MRW  |          |
| 0x306          | 001100000110  | mcounteren | M态计数器使能                       | MRW  | 只读0    |
| 0x340          | 001101000000  | mscratch   | M态Trap处理程序寄存器               | MRW  |          |
| 0x341          | 001101000001  | mepc       | M态异常PC                           | MRW  |          |
| 0x342          | 001101000010  | mcause     | M态Trap原因                         | MRW  |          |
| 0x343          | 001101000011  | mtval      | M态Trap时出错地址或指令             | MRW  |          |
| 0x344          | 001101000100  | mip        | M态中断待处理                       | MRW  |          |
| 0xb00          | 101100000000  | mcycle     | M态当前处理器运行周期数             | MRW  |          |
| 0xb02          | 101100000010  | minsret    | M态当前指令提交数                   | MRW  |          |
| 0x7a0          | 011110100000  | tselect    | 调试/跟踪触发寄存器选择             | MRW  | 只读0    |
| 0x7a1          | 011110100001  | tdata1     | 第一个调试/跟踪触发数据寄存器 | MRW  | 只读1    |

这些寄存器乍一看数量很多，但许多寄存器可以实现为Read Only常量，且 **写时不产生异常** 。

（写异常只判断其本身的权限，例如mvendorid不可写需要在写时产生异常，但mcounteren在规定上是可写，写时不产生异常，但不改变其值。）

这些寄存器具体的每个field大家可以自行查阅[https://five-embeddev.com/quickref/csrs.html](https://five-embeddev.com/quickref/csrs.html)。其中mstatus可能包含许多大家不了解的field，未实现的功能可直接填0，例如mprv可直接忽略，而依赖mprv的mpp自然也可忽略。而如果写入CSR时写入了未支持的特性，需要在提交给CSR时将对应field维持不变，以便软件用于检测处理器是否支持某个硬件特性。

还需要提醒的是，cycle和instret可以实现为mcycle和minstret的只读副本，且由于这两个寄存器是否可在User Mode读取取决于mcounteren的设置，但由于mcounteren我们已经设置为只读0，因此我们在CSR检查上，可简单实现为只要在User Mode使用CSR指令直接产生Illegal Instruction异常即可。

### CSR地址格式

CSR地址一共有12个二进制位，设计Decoder与CSR执行模块所需要考虑的编码格式如下：

#### CSR是否可写的判断

在RISC-V指令集中，`CSR_Address[11:10]`表示用途与是否可写。其中，取值为`00`,`01`,`10`时表示为可读写，取指为`11`时表示只读。因此，我们可通过判断这两位是否为`11`来判断CSR是否可写，这涉及到异常处理。

#### CSR可访问权限的判断

在RISC-V指令集中，`CSR_Address[9:8]`表示该CSR寄存器可被访问的特权级。使用`00`表示User Mode，`01`表示Supervisor Mode（我们不实现），`11`表示Machine Mode。其中，当一个CSR可被某个特权级访问时，更高的特权级也可以访问它。

例如`cycle`寄存器地址为`0xc00`，二进制地址下`[9:8]`部分为0，因此是一个User Mode CSR，在User Mode、Supervisor Mode与Machine Mode均可以访问。

!!! info
    等学习了特权指令后，可以考虑观察CSR操作指令中CSR地址编码的位置，以及各特权级下的特权指令编码，你有什么发现？这一发现可以如何简化你的Decoder设计？

## 异常处理

### 异常类型

针对ISA为RV64IMA+U Mode的处理器（符合Linux NOMMU运行需求），我们需要实现的异常如下：

| 异常号 | 异常描述                       | 中文                         | 五级流水线中推荐出现的阶段 |
| ------ | ------------------------------ | ---------------------------- | -------------------------- |
| 0      | Instruction address misaligned | 取指PC非对齐                 | IF、EXE                    |
| 1      | Instruction access fault       | 取指访问异常                 | IF                         |
| 2      | Illegal instruction            | 非法指令                     | ID、EXE（CSR指令）         |
| 3      | Breakpoint                     | 断点                         | ID                         |
| 4      | Load address misaligned        | 内存Load非对齐               | EXE（也可以是MEM）         |
| 5      | Load access fault              | 内存Load访问异常             | MEM                        |
| 6      | Store/AMO address misaligned   | 内存Store/原子操作地址非对齐 | EXE（也可以是MEM）         |
| 7      | Store/AMO access fault         | 内存Store/原子操作访问异常   | MEM                        |
| 8      | Environment call from U-mode   | 在User Mode使用ECALL指令     | ID                         |
| 11     | Environment call from M-mode   | 在Machine Mode使用ECALL指令  | ID                         |

其中，Breakpoint异常在硬件没有实现Watchpoint时，只在使用`EBREAK`指令时产生，为此我们可以在ID阶段判断。

对于Environment call from U-mode与Environment call from M-mode两个异常，我们只在使用`ECALL`指令时产生，具体产生哪个取决于当前特权级，由于我们在ID阶段也需要判断是否为M-Mode来处理是否可执行特权指令，因此建议将当前特权级信息从CSR接入Decoder，直接由Decoder产生。

对于Illegal instruction中有两个来源的问题，是因为RISC-V要求使用CSR指令访问不存在的CSR或权限检测不通过时（包含是否可写以及是否可以在当前特权级访问），需要触发Illegal instruction异常，因此Illegal instruction异常这里有两个产生的流水线阶段，一个是ID阶段指令译码失败，另一个是CSR访问时检查失败。

对于Instruction address misaligned有两个来源，是因为除了取指本身PC非对齐外，还需要考虑Branch/Jump指令本身导致PC非对齐的问题。有的同学可能会想到既然已经不允许跳转到非对齐指令，为什么该异常还会出现在IF，这是因为我们使用异常处理返回也可以跳转到该PC地址，而mret等指令并不检查返回的epc。

其他异常的实现相信大家看了描述可以自行理解。

### 访存地址非对齐

RISC-V Specification并不规定CPU是否需要支持非对齐访存，对于不支持非对齐访存的CPU，当访存不对齐的时候可以产生异常，通常操作系统/SBI等软件会增加非对齐访存的处理模块。

建议同学们的CPU不支持非对齐访存以简化Cache设计，如果你的Cache实现了块多字，也可以实现为只有跨Cache Line时才产生该异常，以提高出现非对齐访存的情况的性能。

### Trap Value

部分异常产生时还需要写入Trap Value来告知软件一些信息。

涉及到写入Trap Value的情况如下：

- Instruction address misaligned、Instruction access fault
  - 当由Branch/Jump指令产生时，Trap Value为跳转后的非对齐地址
  - 当由IF阶段产生时（例如mret时mepc非对齐），Trap Value为PC
- Load address misaligned、Load access fault：Trap Value为访存地址
- Store/AMO address misaligned、Store/AMO access fault：Trap Value为访存地址
- 其他异常：Trap Value写0

### 异常处理过程

对于只有M Mode的处理器，当异常发生时，硬件处理流程如下：

```verilog
// 保存trap_value与trap_cause，由异常本身产生
CSR[mtval] <= trap_value;
CSR[mcause] <= trap_cause;
// 记录异常时的PC
CSR[mepc] <= PC;
// 关闭中断，并记下之前的中断是否使能到mpie
CSR[mstatus].mpie <= CSR[mstatus].mie;
CSR[mstatus].mie <= 0;
// 保存当前特权级，以便后续使用MRET能够返回到正确特权级
CSR[mstatus].mpp <= current_privileged_mode;
// 将当前特权级切换到Machine_Mode，毕竟异常处理代码需要运行在该模式
current_privileged_mode <= Machine_Mode;
// 根据CSR[mtvec]跳转到异常处理PC处
//（同学们可以思考RTL如何实现，PC寄存器可不在CSR里，可以思考如何作为另一个跳转源）
PC <= {CSR[mtvec].base,2d'0} + (mtvec->mode ? mcause.cause * 4 : 0); // 注：mtvec有两种mode，具体可以看该寄存器的说明
```

### 异常返回流程

见后续特权指令的`MRET`部分。

## 特权指令

### ZiCSR

CSR读写指令如下：

```
                               funct3            opcode
+-------------------+-----------+-----+---------+---------+
| CSR Address[11:0] |  rs1[4:0] | 001 | rd[4:0] | 1110011 | CSRRW
+-------------------+-----------+-----+---------+---------+
| CSR Address[11:0] |  rs1[4:0] | 010 | rd[4:0] | 1110011 | CSRRS
+-------------------+-----------+-----+---------+---------+
| CSR Address[11:0] |  rs1[4:0] | 011 | rd[4:0] | 1110011 | CSRRC
+-------------------+-----------+-----+---------+---------+
| CSR Address[11:0] | uimm[4:0] | 101 | rd[4:0] | 1110011 | CSRRWI
+-------------------+-----------+-----+---------+---------+
| CSR Address[11:0] | uimm[4:0] | 110 | rd[4:0] | 1110011 | CSRRSI
+-------------------+-----------+-----+---------+---------+
| CSR Address[11:0] | uimm[4:0] | 111 | rd[4:0] | 1110011 | CSRRCI
+-------------------+-----------+-----+---------+---------+
 31               20 19       15 14 12 11      7 6       0
```

!!! warning
    注：OPCODE等于`0b1110011`时，在RISC-V ISA中定义为`SYSTEM`，当funct3取指不为以上6个数时（还有2个数可取），他们不代表CSR指令，需要进一步decoder。

其中，指令行为如下所示：

!!! info
    注：本表述其实省略了CSR只写不读时带来的副作用的说明。`CSRRW`与`CSRWRI`指令按照RISC-V文档的严格定义，需要在`rd`寄存器号为`0`时不进行CSR读操作（写操作依然保留），但我们所实现的CSR寄存器范畴不存在这样的情况，同时目前所涉及CSR的异常不存在只写不触发，读写就要触发的情况，因此不会影响到其异常处理行为或是处理器行为，因此没有特殊标注。

#### CSRRW

将`CSR[addr]`当前值读取到`GPR[rd]`，然后将新值从`GPR[rs1]`写入`CSR[addr]`。

```verilog
GPR[rd] <= CSR[csr_addr];
CSR[csr_addr] <= GPR[rs1];
```

#### CSRRS

将`CSR[addr]`当前值读取到`GPR[rd]`，然后将`GPR[rs1]`中二进制为1的位置在`CSR[addr]`中置1。

```verilog
GPR[rd] <= CSR[csr_addr];
if (rs1 != x0) CSR[csr_addr] <= CSR[csr_addr] | GPR[rs1];
```

#### CSRRC

将`CSR[addr]`当前值读取到`GPR[rd]`，然后将`GPR[rs1]`中二进制为1的位置在`CSR[addr]`中置0。

```verilog
GPR[rd] <= CSR[csr_addr];
if (rs1 != x0) CSR[csr_addr] <= CSR[csr_addr] | GPR[rs1];
```

#### CSRRWI, CSRRSI, CSRRCI

分别与上述三个不带I后缀的指令相同，只是将`GPR[rs1]`替换为`uimm`（无符号扩展那5位到64位）。

然后在判断是否写行为上，将`if (rs1 != x0)`替换为`if (uimm != 0)`。

#### 异常处理

CSR指令在以下情况产生`Illegal instruction`异常：

1. 操作的CSR所属特权级大于当前特权级。（对于五级流水可考虑在ID阶段判断）

    例如在User Mode操作Machine Mode CSR属于此类。

    CSR所属特权级可从其地址的`[9:8]`部分判断，前文已介绍。

2. CSR不可写，但CSR操作指令包含了写行为。（对于五级流水可考虑在ID阶段判断）

    CSR是否可写可从CSR地址的`[11:10]`部分判断，前文已介绍。
    
    如果此时指令的rs1为x0（ **注意如果寄存器不是x0即使内容为0则代表写0，不可视为不写，所涉及异常依旧需要产生 ** ）或uimm部分为0，则不认为处理器包含写CSR行为。

    否则，若产生写CSR行为，如果寄存器不可写，则产生Trap。

3. 所访问的CSR地址未在处理器中实现（对于五级流水可考虑在EXE阶段判断）

4. 写CSR时写入了Write Only Legal Values的位域（可选，不推荐实现）

    在RISC-V手册中，定义了一些具体CSR寄存器中的部分域为Write Only Legal Values，处理器实现可以在写入这些域时产生`Illegal instruction`异常， **也可以不产生** 。
    
    建议不产生以简化处理器设计难度。标准的软件（如Linux, OpenSBI）都能够适应这两种实现的处理器的行为。

### 异常返回指令

异常返回指令如下：

```
  funct7                   funct3          opcode
+---------+-------+--------+-----+-------+---------+
| 0011000 | 00010 |  00000 | 000 | 00000 | 1110011 | MRET
+---------+-------+--------+-----+-------+---------+
 31     25 24   20 19    15 14 12 11    7 6       0
```

mret硬件执行流程如下：

```verilog
// 恢复中断使能，并默认mpie为1
CSR[mstatus].mie <= CSR[mstatus].mpie;
CSR[mstatus].mpie <= 1;
// 跳转到异常发生前特权级
current_privileged_mode <= CSR[mstatus].mpp;
// 检查上次异常产生的特权级是否非M_MODE，若非，则关闭修改特权级功能
if (CSR[mstatus].mpp != User_Mode) CSR[mstatus].mprv <= 0;
CSR[mstatus].mpp <= User_Mode;
// 返回之前的PC（注：该PC可能被软件改为PC+4，例如使用ECALL指令做系统调用时）
PC <= CSR[mepc];
```

异常：

- 如果指令为MRET，则在当前特权级并非M-Mode时，产生Illegal instruction异常。

注：如果因为mepc不对齐导致在返回时产生取指非对齐异常，则产生异常的mepc

### 中断管理指令

这里涉及一条指令：WFI（Wait For Interrupt）

```
  funct7                   funct3          opcode
+---------+-------+--------+-----+-------+---------+
| 0001000 | 00101 |  00000 | 000 | 00000 | 1110011 | WFI
+---------+-------+--------+-----+-------+---------+
 31     25 24   20 19    15 14 12 11    7 6       0
```

用于一些有电源管理的处理器暂时关闭等待下一次中断的产生，我们写的FPGA上运行的CPU直接把该指令当做空指令即可。

异常：

- 如果该指令在用户特权级运行，需要产生异常。


### ECALL, EBREAK

产生对应的异常即可，见前面的`异常处理类型`的描述。