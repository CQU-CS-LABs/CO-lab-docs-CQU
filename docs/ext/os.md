# 运行操作系统

## 确保你能够使用AXI接口的SoC

由于操作系统运行需要较大内存支持，而SRAM无法做到高频率大容量，且我们使用的FPGA的SRAM总容量也只有512KB，因此同学们需要首先将CPU连接上AXI。

如果只想仿真使用，可以通过我们发的基于[soc-simulator/sram](https://github.com/cyyself/soc-simulator/tree/nscscc_sram_func)版本，在SRAM接口的CPU上完成TLB的支持以及OS的调试。

## TLB支持

对于运行带虚拟内存支持的操作系统（例如uCore与Linux），首先需要添加TLB模块，这一点请看[MMU](../TLB.md)章节。

## 运行PMON相比功能测试的新增需求

1. Cache指令支持（没有Cache可实现为NOP，所有种类的Cache指令可简单实现为Write Back+Invalidate）
2. MUL指令支持

## 单纯运行ucore-thumips相比功能测试的新增需求

1. TLB支持（含TLB指令，且取指/Load/Store能够在虚拟地址执行）。
2. CP0中ebase的支持（请自行参考MIPS文档）

## 运行主线Linux相比功能测试的新增需求

1. TLB支持
2. CP0完整支持（MIPS32文档中所有的Required以及和TLB/Cache有关的所有功能）。
3. 除Branch-Likely外的所有MIPS32整数指令支持（Branch-Likely可以手动关，其他指令关掉不那么容易）

除此之外，运行Linux的难点在于：

1. 延迟槽可能有一些非常特殊的指令，会带来较多Bug。
2. 新增的大量指令可能没有经过详细测试，尤其是非对齐访存非常容易写错。

## 编译环境的安装

调试OS的运行离不开修改OS的代码，因此需要大家有自己编译OS代码的能力，以下材料所涉及的软件编译均可以通过安装Debian衍生的Linux发行版中的`gcc-mipsel-linux-gnu`软件包进行：

```shell
apt install gcc-mipsel-linux-gnu
```

## 可以参考/复现的材料：

SoC上板工程：

1. [CDIM-SoC-n4ddr](https://github.com/cyyself/CDIM-SoC/tree/n4ddr_porting)

    这是CQU Dual Issue Machine的SoC工程，使用Vivado Block Design搭建，同学们可以打开学习SoC内部构造，并可以尝试加入其核心代码，然后烧入U-Boot运行。

2. [EggMIPS-SoC](https://github.com/cyyself/EggMIPS-SoC)

    这是龙芯杯官方提供的运行OS工程的N4DDR移植版本。并在仓库中提供了编译好的PMON BIOS可以直接使用。如你有PMON的修改/调试需求，我们的建议是不要花时间折腾这个老古董，直接使用现代的U-Boot。

SoC仿真：

1. [soc-simulator/nscscc_cdim](https://github.com/cyyself/soc-simulator/tree/nscscc_cdim)

    CQU Dual Issue Machine比赛期间使用soc-simulator做的测试框架，需要将CPU封装为AXI接口，大家可以阅读该代码参考运行不同的软件所需要的SoC设计。

    该Git仓库涉及软链接文件，除了需要下载[CEMU/mips32-nscscc](https://github.com/cyyself/cemu/tree/mips32-nscscc)分支的代码外，还需注意如使用WSL请勿在`/mnt`路径下进行git clone，因为`/mnt`不支持软链接。

CPU模拟器：

1. [CEMU/mips32-nscscc](https://github.com/cyyself/cemu/tree/mips32-nscscc)

    同学们可以查看`src/main.cpp`了解一个软件模拟器如何运行Linux，并对该模拟器进行修改符合你的CPU行为进行差分测试/Debug。

系统软件：

1. [ucore-thumips](https://github.com/cyyself/ucore-thumips)

    uCore操作系统到MIPS的移植，直接使用make即可编译，然后可以使用龙芯的[QEMU](https://gitee.com/loongsonlab/qemu)，直接使用`make qemu`运行。
    
    也可以使用[CEMU/mips32-nscscc](https://github.com/cyyself/cemu/tree/mips32-nscscc)运行`ucore-kernel-initrd.bin`，具体可看代码了解文件应该放置什么路径以及代码如何修改。

    仿真运行已在先前介绍，上板运行取决于你使用的Bootloader。

2. [u-boot-cdim_soc](https://github.com/cyyself/u-boot/tree/cdim_soc)

    可以观察commit，了解如何写U-Boot设备树，配置栈地址等保证不与内核冲突，同时需注意U-Boot不会从CP0的config读取缓存行大小，需手动在配置文件中指定。

3. [linux-cdim_soc](https://github.com/cyyself/linux/commits/cdim_soc)

    可以观察commit，了解如何写Linux设备树，配置CPU有关参数等。


## 实战建议

1. 首先确保你有一个AXI接口的CPU，并完成了TLB的添加。

    TLB测试可暂时跳过，调试操作系统时再测试TLB功能，通过龙芯杯的测试不代表你的TLB添加正确（因为它不测试虚拟地址实际取指/访存），不通过这个测试也不代表你的TLB实现不正确（它要求实现32项，但在32项TLB的情况下频率很难有保证）。

    尽管SRAM接口加了TLB也可以在仿真条件下运行操作系统，但后续改AXI涉及到Cache和取指访存的几乎推翻重写，会带来更多的工作量。

2. 使用[ucore-thumips](https://github.com/cyyself/ucore-thumips)+[soc-simulator/nscscc_cdim](https://github.com/cyyself/soc-simulator/tree/nscscc_cdim)完成uCore的调试。

3. 使用[CDIM-SoC-n4ddr](https://github.com/cyyself/CDIM-SoC/tree/n4ddr_porting)和[u-boot-cdim_soc](https://github.com/cyyself/u-boot/tree/cdim_soc)完成上板。

4. 继续完成更多指令/功能的添加，然后调试运行Linux。


## CQU Dual Issue Machine 仿真运行 uCore 演示

```shell
git clone https://github.com/Maxpicca-Li/CDIM.git
cd CDIM
git clone https://github.com/cyyself/soc-simulator.git -b nscscc_axi
git clone https://github.com/cyyself/ucore-thumips.git
cd ucore-thumips
make -j `nproc`
cd .. # at CDIM
cd soc-simulator
sed -i 's/..\/..\/mycpu/..\/mycpu/g' Makefile # 修改Makefile中的SRC_DIR由../../mycpu到../mycpu
make
./obj_dir/Vmycpu_top -ucore
```

你将会看到以下输出：

```shell
➜  soc-simulator git:(nscscc_axi) ✗ ./obj_dir/Vmycpu_top -ucore
++setup timer interrupts
Initrd: 0x8006aa50 - 0x800c224f, size: 0x00057800, magic: 0x2f8dbe2a
(THU.CST) os is loading ...

Special kernel symbols:
  entry  0x80000108 (phys)
  etext	0x8002B400 (phys)
  edata	0x800C2250 (phys)
  end	0x800C5570 (phys)
Kernel executable memory footprint: 617KB
memory management: buddy_pmm_manager
memory map:
    [80000000, 82000000]

freemem start at: 80106000
free pages: 00001EFA
## 00000020
check_alloc_page() succeeded!
check_pgdir() succeeded!
check_boot_pgdir() succeeded!
-------------------- BEGIN --------------------
--------------------- END ---------------------
check_slab() succeeded!
kmalloc_init() succeeded!
check_vma_struct() succeeded!
check_pgfault() succeeded!
check_vmm() succeeded.
sched class: RR_scheduler
ramdisk_init(): initrd found, magic: 0x2f8dbe2a, 0x000002bc secs
sfs: mount: 'simple file system' (81/6/87)
vfs: mount disk0.
kernel_execve: pid = 2, name = "sh".
user sh is running!!!
$ 
```