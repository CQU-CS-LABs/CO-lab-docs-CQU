## RISC-V指令集简介

RISC-V是一个基于精简指令集的开源指令集架构（ISA），与大多数指令集相比，RISC-V指令集可以自由地用于任何目的，允许任何人设计、制造和销售RISC-V芯片和软件而不必支付给任何公司专利费，因此非常适用于我们的教学活动。

### 优势

相比于MIPS，对于硬件综合设计课程具有以下优势：

1. 没有延迟槽，包括跳转、异常在内的刷新逻辑统一化，对于复杂架构的设计极其友好。
2. 具有可裁剪性，实现一个基本RV32I仅40条指令，甚至不需要实现乘除法（由编译器使用其他指令模拟），就可以运行C语言程序，对于64位则需要55条。
3. Linux具有NOMMU的支持，不需要实现MMU（TLB+PTW）即可运行Linux（至少实现 RV-IMA + CSR + fence_i）。

### 劣势

相比MIPS，对于硬件综合设计课程具有以下劣势：

1. 如果实现MMU，除实现TLB外还需要实现硬件页表遍历，且需要实现一系列非常复杂的Supervisor特权级的CSR，较为复杂。
    
    但考虑到不实现MMU也能运行Linux，最终达成运行Linux目标比MIPS简单。

### 配置环境

1. 准备环境

    根据你的发行版进行不同的操作：

    - Ubuntu
        ```shell
        sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev ninja-build
        ```

    - Fedora
        ```shell
        sudo yum install autoconf automake python3 libmpc-devel mpfr-devel gmp-devel gawk  bison flex texinfo patchutils gcc gcc-c++ zlib-devel expat-devel
        ```

    - Arch Linux
        ```shell
        sudo pacman -Syyu autoconf automake curl python3 libmpc mpfr gmp gawk base-devel bison flex texinfo gperf libtool patchutils bc zlib expat
        ```

    - macOS + Homebrew

        ```shell
        brew install python3 gawk gnu-sed gmp mpfr libmpc isl zlib expat
        brew tap discoteq/discoteq
        brew install flock
        ```

2. 下载riscv-gnu-toolchains

    ```shell
    git clone https://github.com/riscv/riscv-gnu-toolchain
    ```

3. 编译工具链

    - 如果你不想实现乘除法指令（这样编译出来的编译器会将乘除法使用软件实现）：

        ```shell
        cd riscv-gnu-toolchain
        ./configure --prefix=/opt/riscv --with-arch=rv64ia --with-abi=lp64
        make linux -j `npoc`
        ```

    - 如果你想实现乘除法指令：

        ```shell
        cd riscv-gnu-toolchain
        ./configure --prefix=/opt/riscv --with-arch=rv64ima --with-abi=lp64
        make linux -j `npoc`
        ```

!!! info

    Tips: 你也可以修改--prefix参数安装两份编译器，一份软乘除法，一份硬乘除法。

!!! info

    里面涉及到的`/opt/riscv`文件夹应提前建立，可以这么做：
    ```shell
    sudo mkdir -p /opt/riscv
    sudo chown $USER /opt/riscv
    ```


### 框架

```shell
git clone https://github.com/CQU-CS-LABs/CO-LAB-RISCV.git
```

!!! warning

    默认指令集为RV64ima，如果没有实现乘除法请自行修改`test/test_workbench/soft/Makefile`。

!!! info
    
    使用框架需要将编译器产生的bin文件夹加入PATH环境变量中（使用WSL的同学需要在WSL内加入而不是Windows中加入），对于 **当前终端** 添加，可以这么做：`export PATH=/opt/riscv/bin:$PATH`

### 流程

1. 熟悉框架、熟悉RV64IM指令集、熟悉CSR。

2. 完成RV64I指令集，跑通框架中提供的Hello World测试。

    （注意编译器编译时的选项，如果没有实现乘除法请使用--with-arch=rv64ia）

3. 实现CSR与乘除法（RV64M），跑通框架中提供的RISC-V Tests。

4. 实现你想实现的任何内容，例如A扩展、Cache、分支预测，或是上板SoC调通Linux等。