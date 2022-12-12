# RISC-V 特权级介绍

相信大家已经在操作系统课程中对特权级有了基础的了解，我们在实现特权级还需要实现特权指令以及CSR寄存器，以及异常处理的逻辑。

这一部分是对[RISC-V Privileged Spec](https://riscv.org/technical/specifications/)中我们实验需要用到的部分进行精炼，精炼到只需要实现一些简单的逻辑，并最终能完成运行Linux NOMMU的目标，同学们课后有时间建议直接阅读英文Spec。


## 特权等级

RISC-V分为以下几个特权等级：

| 等级编号 | 名字             | 简称 | 实现难度     | 目标建议                             |
| -------- | ---------------- | ---- | ------------ | ------------------------------------ |
| 0        | User/Application | U    | ☆            | 想要运行U-Boot+Linux NOMMU的同学实现 |
| 1        | Supervisor       | S    | ☆☆☆          | （课程期间很难做出来）               |
| 2        | Hypervisor       |      | ☆☆☆☆☆        | （课程期间几乎不可能做出来）         |
| 3        | Machine          | M    | （必须实现） | 必须实现，且默认在M-Mode运行         |

## CSR寄存器

实现单核运行Linux NOMMU目标，我们需要实现以下CSR寄存器（其实实现异常处理也只有这些寄存器）：

| 地址（16进制） | 地址（2进制） | 名称     | 用途 |
| -------------- | ------------- | -------- | ---- |
|                |               | mstatus  |      |
|                |               | misa     |      |
|                |               | mie      |      |
|                |               | mtvec    |      |
|                |               | mscratch |      |
|                |               | mepc     |      |
|                |               | mcause   |      |
|                |               | mtval    |      |
|                |               | mip      |      |
|                |               | mcycle   |      |

对于没有实现的寄存器，我们可以实现为读时返回0，写时忽略。你也可以选择触发非法指令异常，但会增加实现难度，我们使用的系统软件（包括Linux、U-Boot）都会兼容这两种可行的实现。

## 内存管理

以下列出三个即使达成运行Linux NOMMU也不需要实现的功能，留给同学们后续观看，但同学们有必要花一些时间了解有关功能的概述。

### 物理内存属性（Physical Memory Attributes，PMA）

我们在Cache章节中已经提到，RISC-V指令集上，物理地址的属性（包括是否Uncached、是否可以进行原子操作）由PMA确定，根据RISC-V Privileged Spec规定，硬件可以实现为固定PMA，因此建议大家实现为硬件固定（我们可以按照地址的31位是否为1判断是Cached访问还是MMIO），并按照Cache章节中的SoC地址空间规范编写代码。

### 物理内存保护（Physical Memory Protection，PMP）

为了节省大家有限的时间，不建议课程期间实现PMP，不实现并不影响大家运行Linux NOMMU。有兴趣的同学可以自行阅读Spec。

PMP提供了一些物理地址的轻量级内存保护，保护的范围包括2的n次方对齐的地址或者使用两个PMP寄存器表示一段范围，如果越权访问根据读写类型产生Access Fault异常。

### 基于页表的虚拟内存

实现虚拟内存之前，你需要实现以下功能：

- Supervisor特权级的所有必须实现的CSR（包括实现Trap Virtual Memory、Modified Privilege等功能）

你可以选择以下2种实现：

- 阅读Spec，选择一种或多种虚拟内存类型（RV64推荐至少实现Sv39），实现硬件Page Table Walker填充TLB
  - 需要妥善处理以下问题：
    - 页表遍历器与D Cache的缓存一致性问题（推荐在D Cache中实现PTW填充TLB含iTLB，访存TLB Miss直接处理，取指TLB Miss可以传递信号到Memory阶段，完成PTW后重新执行这条指令或产生Page Fault）
    - 对于实现了压缩指令扩展（C）时需考虑一条指令跨页面的问题（推荐通过一个Buffer解决，与跨Cacheline类似）
- 硬件不实现PTW，添加自定义TLB维护指令，魔改软件
  - 魔改SBI通过Trap Virtual Memory实现类似MIPS/LoongArch32的软件定义页表
  - 每次TLB Miss时产生一个自定义的异常交给SBI完成Page Table Walk填充TLB
  - 这种做法可以对S-Mode透明，不影响Linux运行

由于实现Supervisor特权级的所有必须实现功能已经相当困难，助教在此推荐不实现虚拟内存，如果硬综期间有时间或者课后有兴趣的同学可以自行阅读Spec实现。

在不实现虚拟内存的情况下，你无法运行常规带MMU的Linux，但Linux支持了RISC-V架构下的NOMMU，将**地址无关**的用户程序直接跑在物理内存（这种情况下用户程序是无法使用mmap等系统调用的），在没有PMP的情况下，用户程序也不会有任何内存保护，可以随意修改Linux内核的地址空间，但至少它能工作。

<!-- TODO --->